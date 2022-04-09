---
weight: 50
---

# Versioning

LSUClient uses a three-part version number MAJOR.MINOR.PATCH and follows [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html).

This means you can generally expect all your scripts and integrations to keep working with any one major version (such as 1.x.x).

There is only ever one current release of LSUClient, there is no parallel maintenance of older releases or prior versions.
As soon as a new release is out the prior one is obsoleted.

## What is and isn't covered by the semantic versioning promise

Semantic versioning communicates changes in a softwares "public API".

For the purpose of a PowerShell Module, the public API is the exported functions (cmdlets), classes and the returned objects.

Explicitly exempt from this public API are:

- Private functions
- Internal classes
- Hidden properties
- Output streams other than the success stream (1)

The data structure, type, name, content and behavior of these may change at any point without notice as they are
only intended for internal use by LSUClient itself, or in the case of non-success output streams such as Verbose and Debug,
only for logging and "human consumption" and not to support scripting workloads.

## Minor breaking changes in minor versions

I must admit, I *sometimes* make exceptions from this rule for breaking changes I consider to be "very very minor" in that:

1. I feel they are likely not going to impact any or only very few users
2. They are trivial to adjust for, as in the change(s) required to get everything working again with the new version are very very small

An example of this is the change of the type of the `URL` property on the `[LenovoUpdate]` objects from `[System.Uri]` to `[System.String]` with Version 1.3.0.
So if you were accessing a property or method unique to the `[System.Uri]` object, for example:

```powershell
# Pretend-script written for LSUClient 1.2.5
$OneUpdate = Get-LSUpdate -All | Select-Object -First 1
if ($OneUpdate.URL.Host -like "*.com") {
    Write-Output "Hey! This update was sourced from a .com domain!"
} else {
    Write-Output "Some other domain!"
}
```

that would have broken because once `$OneUpdate.URL` became a string, it no longer had a `Host` property.

This means in the code snippet above `$OneUpdate.URL.Host` will evaluate to `$null` and either always print `Some other domain!`
or error with `The property 'Host' cannot be found on this object.` if you are running in PowerShell Strict Mode.

However, a quick fix to get the same snippet working again could be to just cast `$OneUpdate.URL` back to `[System.Uri]`:

```powershell {hl_lines=[3]}
# Pretend-script updated for LSUClient 1.3.0+
$OneUpdate = Get-LSUpdate -All | Select-Object -First 1
if (([Uri]$OneUpdate.URL).Host -like "*.com") {
    Write-Output "Hey! This update was sourced from a .com domain!"
} else {
    Write-Output "Some other domain!"
}
```

Any such "minor breaking change" will be noted in the release notes, and will also be accompanied by a bump of the MINOR version number.

