# 11.0 Upgrade Guide

### Introduction

FluentValidation 11.0 is a major release that included several breaking changes. Please review this document carefully before upgrading from FluentValidation 10.x to 11.

There were 2 main goals for this release:
- Removing deprecated code and support for obsolete platforms
- Update sync-over-async workflows to clearly throw an exception

Below is a summary of all the changes in this release:

### Changes in supported platforms

- Support for .NET Core 2.1 has been removed, as Microsoft support has ended for this release.
- Support for the `netstandard2.0` platform has been removed removed from the core library. `netstandard2.1` is now the minimum supported version. This implicitly means that FluentValidation 11.x can't be used on legacy .NET Framework 4.x. If you need to run on .NET 4.x or another netstandard2 platform, you must use FluentValidation 10.x instead.

### Sync-over-async now throws an exception

In FluentValidation 10.x and older, if you attempted to run an asynchronous validator synchronously, the asynchronous rules would silently be run synchronously. This was unintutive and would lead to deadlocks. 

Starting in FluentValidation 11.0, validators that contain asynchronous rules will now throw a `AsyncValidatorInvokedSynchronouslyException` if you attempt to invoke them synchronously. You must invoke these validators asynchronously.

This affects rules that contain any of the following:
- Calls to `MustAsync`
- Calls to `WhenAsync`\`UnlessAsync`
- Calls to `CustomAsync`
- Use of any custom async validators 

### OnFailure and OnAnyFailure removed

The deprecated methods `OnFailure` and `OnAnyFailure` have been removed.

These were callbacks that could be used to define an action that would be called when a particular rule fails. These methods were deprecated in 10.x as they allowed the standard FluentValidation workflow to be bypassed, and additionally they have caused various maintenance issues since they were introduced. 

### Test Helper changes

The deprecated extension methods `validator.ShouldHaveValidationErrorFor` and `validator.ShouldNotHaveValidationErrorFor` have been removed. The recommended alternative is to use `TestValidate` instead, [which is covered in the documentation here](https://docs.fluentvalidation.net/en/latest/testing.html).

### ASP.NET Core Integration changes

The deprecated property `RunDefaultMvcValidationAfterFluentValidationExecutes` within the ASP.NET Configuration has been removed. 

If you were making use of this property, you should use `DisableDataAnnotationsValidation` instead. Note that this property is the inverse of the previous behaviour:

```csharp
// Before:
services.AddFluentValidation(fv => {
  fv.RunDefaultMvcValidationAfterFluentValidationExecutes = false;
});

// After:
services.AddFluentValidation(fv => {
  fv.DisableDataAnnotationsValidation = true;
});

```

### Removal of backwards compatibility property validator layer

The non-generic `PropertyValidator` class (and associated classes/helpers) have been removed. These classes were deprecated in 10.0. If you are still using this class, you should migrate to the generic `PropertyValidator<T,TProperty>` instead. 

### Internal API Changes

Several of the methods in the Internal API have been removed. These changes don't affect use of the public fluent interface, but may impact library developers or advanced users.

- `IValidationRule<T,TProperty>.CurrentValidator` has been removed (use the `Current` property instead)
-`IValidationRule<T,TProperty>.Current` now returns an `IRuleComponent<T,TProperty>` interface instead of `RuleComponent<T,TProperty>` (necessary to support variance) 
-`IValidationRule<T,TProperty>.MessageBuilder`'s argument is now an `IMessageBuilderContext<T,TProperty>` interface instead of `MessageBuilderContext<T,TProperty>` class (necessary to support variance)
- `IValidationRule<T,TProperty>.MessageBuilder` is now set-only, and has no getter exposed (needed to support variance), meaning you can only have one message builder per rule chain. 
- `IRuleComponent<T,TProperty>.CustomStateProvider` is now set-only to support variance
- `IRuleComponent<T,TProperty>.SeverityProvider` is now set-only to support variance
- `GetErrorMessage` is no longer exposed on `IRuleComponent<T,TProperty>`
- Remove deprecated `Options` property from `RuleComponent`
- The `MemberAccessor` class has been removed as it's no longer used