# ChemDraw Development Issues
Welcome to ChemDraw Development Issues, this is a place to note down the ChemDraw development related issues for future reference.

**Contents**
- [Refactoring](#Refactoring)
- [SKU and Component](#SKU-and-Component)

## Refactoring

### SKU and Component

#### How can we model different SKUs in unit-test and test against them?

I met a problem writing unit-test against a function that depends on SKU. If it's `ChemOffice Professional`, it should return a value. If it's `ChemDraw Professional`, it should return another value. The SKU is represented by a global value `sSKUActivated`.

We can consider using the common pattern that we use for injecting global values for unit-test. For example, can just create a helper class `SKUInfo`, and make it a singleton and injectable?

Or, we can move the global variable into the existing class `VersionInfo`, and make this `VersionInfo` a singleton and injectable?

Looks like the class `VersionInfo` is responsible for getting the App Name, Program Name (really?), Versioned Program Name (again?), Activation Prog Name (?), Version Name, App Level Name, App Level, Unified App Name (what a pain?), etc. So, it's natural that it can provide something like `GetActivatedSKU()`, `GetProdCode()`, `GetLevCode()`?

I was trying to write unit-test for these methods:
```
bool AddInManager::AreAddInsSupported() const
{
    // Add-ins are available in ChemDraw Professional and above
    return IsComponentAvailable(Component_CBD_ULTRA);
}

bool AddInManager::AreWebSourceAddInsSupported() const
{
    // The web source add-ins are supported only by this SKU: ChemOffice Professional
    return AreAddInsSupported() && (GetActivatedSKU() == kSKUChemOfficeProfessional);
}
```

From **Steve**:
Functionality should probably depend on features, not SKUs. You can then use FeatureInfoMock to mock the feature options.
Personally I would move all the SKU stuff into FeatureInfo so one class has responsibility for this. VersionInfo should be avoided where possible because on Windows it uses the CLR in an unsafe way which can cause crashes on quit.

These methods need to be rewritten to use features. We shouldn't be directly checking like this (for the exact reason that it is difficult to mock in tests)
