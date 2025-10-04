# Static Registration In C++

Date: 2025 September 30

If you've worked with shared libraries before, you may have seen a pattern where some object in a translation unit (plug-in) registers itself to some registry in a "drop-in"-like manner. This allows for extensions to some interface without needing to know the definition of the extension via header inclusion. I found this idea of registration confusing, but I recently implemented my own version as a learning exercise. My goal was to create some registry that allowed objects to register callbacks implementing some instance-specific functionality. In this case, we'll develop a mechanism to register and refresh cached data. The purpose of this post is to document what I have learned about static registration in C++.

## Calling Functions At File Scope

One aspect of static registration that left me perplexed (among many) was how objects could execute some registration function outside of a block scope (*i.e.* outside a function). Static non-local objects are declared and defined at file or namespace scope; it is not possible to call a function at this scope to register the object like we would in a function. This is especially true if the registration performed via a method of said object.

```cpp
namespace {

// Try to register to some registry

static RegistryObject sMyObject{/* Relevant data */};
// sMyObject.registerObject(/**/); // ERROR. We can't call methods outside block scope.
}
```

However, I realized that in order to execute some function procedure at such a scope, we must take advantage of object construction. Naturally, constructors can execute outside of block scope in order to initialize objects, so that mechanism is the key to registering objects to some registry outside of block scope. Put concisely, any registration logic needs to be in the constructor initializing the static object.

## Guaranteed Static Initialization Order

I already had a little experience with static initialization order in [my previous post](/basics/static-initialization-order), but this adventure brought it to the forefront again. A registry is typically something that should only exist once. In other words, registries are represented as singletons. In order to ensure that all static objects properly register, it is imperative that the registry is initialized first. However, static objects in different translation units have an unspecified initialization order; how do we enforce the desired order? The solution is to take advantage of static local objects.

Unlike nonlocal objects, static local objects are initialized the *first* time program hits the variable's declaration.  As a result, there's no chance of a static local resetting to some default after initialization. We can implement the registry as follows:

```cpp
// To see as an actual minimal project, see: 
// https://godbolt.org/z/9jx4xeadb

#include <cassert>
#include <functional>
#include <map>
#include <stdexcept>
#include <vector>

namespace cache {

// Defines a list of conditions under which a cache should be reset
enum class ResetCondition {
    kFileSystemChanged,
    kClockReset,
    // ...
};

// A helper for registering callbacks
class CacheResetRegistryAgent;

// A Registry of callbacks for resetting cache data.
// Register a callback by instantiating a @c CacheResetRegistryAgent object.
class CacheResetRegistry {
   public:
    static void resetCaches(ResetCondition condition);

   private:
    using CallbackType = std::function<void()>;

    static auto instance() -> CacheResetRegistry&;
    void registerCallback(CallbackType const& callback,
                          std::vector<ResetCondition> const& resetConditions);
    void resetCachesImpl(ResetCondition condition);

   private:
    std::map<ResetCondition, std::vector<CallbackType>> mCallbackMap;

    friend class CacheResetRegistryAgent;
};

class CacheResetRegistryAgent {
   public:
    using CallbackType = CacheResetRegistry::CallbackType;

    CacheResetRegistryAgent(CallbackType const& callback,
                            std::vector<ResetCondition> const& conditions) {
        if (!callback) {
            throw std::invalid_argument("Input callback must be non-empty.");
        }

        CacheResetRegistry::instance().registerCallback(callback, conditions);
    }
};

//---

auto CacheResetRegistry::instance() -> CacheResetRegistry& {
    // The ISOCPP FAQ recommends leaking a heap-allocated object, but it isn't
    // necessary here because no object relies on this registry for their
    // destructor.
    static CacheResetRegistry sRegistry;

    return sRegistry;
}

void CacheResetRegistry::registerCallback(
    CallbackType const& callback,
    std::vector<ResetCondition> const& conditions) {
    for (const auto& condition : conditions) {
        if (mCallbackMap.contains(condition)) {
            mCallbackMap[condition].push_back(callback);
        } else {
            mCallbackMap.insert({condition, {callback}});
        }
    }
}

void CacheResetRegistry::resetCaches(ResetCondition condition) {
    CacheResetRegistry::instance().resetCachesImpl(condition);
}

void CacheResetRegistry::resetCachesImpl(ResetCondition condition) {
    if (mCallbackMap.contains(condition)) {
        for (const auto& callback : mCallbackMap[condition]) {
            callback();
        }
    }
}

}  // namespace cache

namespace {

static int sPizzaCount{5};
static int sBurgerCount{10};

static const cache::CacheResetRegistryAgent kPizzaCacheReset{
    []() { sPizzaCount = 0; }, {cache::ResetCondition::kClockReset}};
static const cache::CacheResetRegistryAgent kBurgerCacheReset{
    []() { sBurgerCount = 0; }, {cache::ResetCondition::kClockReset}};

}  // namespace

int main() {
    using namespace cache;
    
    assert(sPizzaCount == 5);
    assert(sBurgerCount == 10);

    CacheResetRegistry::resetCaches(ResetCondition::kClockReset);
    
    assert(sPizzaCount == 0);
    assert(sBurgerCount == 0);

    return 0;
}
```

From the above example, one can see that static definition of the registry agent objects results in the callbacks' addition to the registry. Calling the registry for a given condition results in the execution of all callbacks registered under that condition, thereby resetting the cache variables. 

## Recap

The mechanism of static registration is achieved by taking advantage of two factors. First, we can execute registration logic at namespace/file scope by leveraging object initialization; constructors enable developers to run code outside of block scope (function) context. Secondly, initialization order of the registry is enforced by using a static local (block scoped) variable. Such variables are guaranteed to be initialized once execution reaches their declaration. As shown in the above example, this can even occur before the execution of `main`. 

To make the process more ergonomic, registration logic is encapsulated into a agent class, making the registration as simple as instantiating that class as a static object.  As the Godbolt implementation exhibits, registration can be done in multiple translation units, even those not attached to an included header, enabling a plug-and-play paradigm for implementers of the registration schema.

## Bonus: Registering with Non-Static Locals

As a challenge, I decided to see if I could make the registration process work for non-static objects as well. The previous design is only valid for objects that live for the duration of the program. Otherwise, the variables captured/used in the callback function(s) could used illegally (dangling reference). Non-local objects could also keep cached information during their lifetimes, and that data may need to be reset during said lifetime.

*Note*: The retelling is considerably more linear than the actual iteration/experimentation. 

If non-static objects are to be supported, the registry needs some mechanism by which "expired" objects are deregistered. This deregistration should be fairly automated; we shouldn't need to perform extensive bookkeeping to support this feature. To make a long story short, I leveraged shared and weak pointers, modifying the interfaces of the registry to use these mechanisms. 

Here is the implementation:

```cpp

#include <concepts>
#include <forward_list>
#include <functional>
#include <iostream>
#include <map>
#include <memory>
#include <optional>
#include <stdexcept>
#include <variant>

namespace cache {
enum class ResetCondition {
    kFileSystemChanged,
    kClockChanged,
    //...
};

// A simple wrapper around a function object
class CacheResetObject {
   public:
    using CallbackType = std::function<void()>;

    CacheResetObject(std::function<void()> const& callback)
        : mCallback(callback) {
        if (!mCallback) {
            throw std::invalid_argument("Callback must be non-empty.");
        }
    }

    void operator()() const { mCallback(); }

   private:
    CallbackType mCallback;
};

class CacheResetRegistrationAgent;

class CacheResetRegistry {
   public:
    static void resetCaches(ResetCondition condition);

   private:
    static auto instance() -> CacheResetRegistry*;
    void registerObject(std::shared_ptr<CacheResetObject> const& resetObject,
                        std::vector<ResetCondition> const& resetConditions);
    void resetCachesImpl(ResetCondition condition);

    std::map<ResetCondition, std::forward_list<std::weak_ptr<CacheResetObject>>>
        mCallbackMap;
    friend class CacheResetRegistrationAgent;
};

class CacheResetRegistrationAgent {
   public:
    using CallbackType = std::function<void()>;
    static_assert(
        std::is_same_v<CallbackType, typename CacheResetObject::CallbackType>);

    CacheResetRegistrationAgent(
        CallbackType const& callback,
        std::vector<ResetCondition> const& resetConditions)
        : mResetObject(std::make_shared<CacheResetObject>(callback)) {
        CacheResetRegistry::instance()->registerObject(mResetObject,
                                                       resetConditions);
    }

   private:
    std::shared_ptr<CacheResetObject> mResetObject;
};

//---

auto CacheResetRegistry::instance() -> CacheResetRegistry* {
    static std::unique_ptr<CacheResetRegistry> sRegistry =
        std::make_unique<CacheResetRegistry>();

    return sRegistry.get();
}

void CacheResetRegistry::registerObject(
    std::shared_ptr<CacheResetObject> const& resetObject,
    std::vector<ResetCondition> const& resetConditions) {
    for (const auto& condition : resetConditions) {
        if (mCallbackMap.contains(condition)) {
            mCallbackMap[condition].push_front(resetObject);
        } else {
            mCallbackMap.insert({condition, {resetObject}});
        }
    }
}

void CacheResetRegistry::resetCachesImpl(ResetCondition condition) {
    if (mCallbackMap.contains(condition)) {
        auto& objectList = mCallbackMap[condition];
        // Remove any "dead" weak_ptrs, The attached object has been destroyed.
        objectList.remove_if([](auto&& item) { return item.expired(); });
        for (const auto& objectPtr : objectList) {
            // We removed all null weak_ptrs, so no need to check output from
            // `lock`
            (*(objectPtr.lock()))();
        }
    }
}

void CacheResetRegistry::resetCaches(ResetCondition condition) {
    CacheResetRegistry::instance()->resetCachesImpl(condition);
}

}  // namespace cache

namespace {

template <typename T, typename Printer = std::ostream>
    requires requires(Printer& os, T const& value) {
        { os << value } -> std::convertible_to<std::ostream&>;
    }
auto operator<<(std::ostream& os, std::optional<T> const& opt)
    -> std::ostream& {
    if (opt) {
        os << opt;
    } else {
        os << "NULLOPT";
    }

    return os;
}
}  // namespace

```