Use C++17 std::filesystem library
==================================

**Julien Marrec, EffiBEM**

 - Original Date: 2020-12-041
 - Revision Date

## Justification for New Feature ##

[#8376](https://github.com/NREL/EnergyPlus/issues/8376) indentified a bug with the `EnergyPlus::FileSystem::moveFile(std::string const &filePath, std::string const &destination)` function on Unix that calls the stdio [`rename`](https://www.cplusplus.com/reference/cstdio/rename/). The rename method would fail when `filePath` and `destination` aren't on the same device (eg: two different hard drives).

Looking into the `FileSystem.cc`, it is ridden with macros to conditionally call into win32 API or unix APIs.

Now that we have enabled C++17, we can leverage `std::filesystem` to simplify things greatly.

Note that there are only a handful of files that use the `FileSystem.hh` header, and by far the main user is `CommandLineInterface.cc`, which will call into the `FileSystem` functions like `getParentDirectoryPath` and use `std::string` concatenation to produce `std::string` representation of paths to be ultimately handled by a system call. Other files that use it mainly make use of `fileExists` or `removeFile` and are very easy to deal with and without much risks.

=> **Bottom line**: the proposed changes mainly affects the CLI.

## E-mail and  Conference Call Conclusions ##

insert text

## Approach ##

There are two things we can do:

1. Only modify the `FileSystem` namespace function to make use of `std::filesystem` internally, but keep providing functions that take in `std::string` arguments and return `std::string` representation of paths.
    * Minor adjustments will be needed to maintain backward compatibility. eg `getParentDirectoryPath` would call `fs::path::parent_path()` then will need to manually append a trailing slash for `std::string` concatenation to work in `CommandLineInterface` (though there are definitely some cases where that's also done in `CommandLineInterface`...).
2. Move everything to use `fs::path`, including in consumers.
    * `CommandLineInterface` would no longer use concatenation of std::string, like `std::string outputFilePrefix = std::string{outDirPathName} + std::string{prefixOutName};`, instead it would use `fs::path` and not have to deal with trailing slashes etc, it can just use the operator/ overload, eg: `fs::path outputFilePrefix = fs::path{outDirPathName} / prefixOutName;`

## Testing/Validation/Data Sources ##

insert text


## References ##

https://en.cppreference.com/w/cpp/filesystem/
https://en.cppreference.com/w/cpp/filesystem/path
