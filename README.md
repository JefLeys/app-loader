# app-loader
An Omnis library to load prerequisite libraries and automatically update libraries under the user's application data folder

## Pre-loading libraries
`app_loader.lbs` manages loading one or more prerequisite libraries before starting a final, primary library. This works by separating prerequisite libraries into a `lib` folder and leaving the final library in `startup`.

Prerequisite libraries will be launched with their `Startup_Task` enabled, but should not rely on any other library to be open. The primary library's `Startup_Task` should be responsible for loading the application.

The expected folder layout is:
```
├── lib
│   ├── prerequisite1.lbs
│   ├── prerequisite2.lbs
│   ├── prerequisite....lbs
│   └── prerequisiteN.lbs
└── startup
    ├── app_loader.lbs
    └── primary.lbs
```
Since Omnis loads libraries in alphabetical order, your primary library should be named to load _after_ `app_loader.lbs`. If this is not doable, rename `app_loader.lbs` to run first by prefixing it with an underscore, i.e. `_app_loader.lbs`.

## Updating libraries
`app_loader.lbs` will also ensure the libraries in the application data directory are the same version as those in the program data directory under `firstruninstall/`. This is accomplished by tracking a **build number**, which is a single incrementing integer denoting one or more libraries has been changed.

When launched, `app_loader.lbs` will compare the `build_number.txt` file in the program directory's `firstuninstall/startup` with `build_number.txt` in the application data's `startup/`. If these build numbers differ the files in `firstruninstall/lib/` and `firstruninstall/startup/` will be copied to the application data directory, overwriting existing files.

Extra files will not be removed.

Any error that occurs will be logged to the Omnis trace log, which will be opened upon receiving an error.

## Live updates
If you have a system to download and update files in the program directory, you can programmatically trigger `app_loader.lbs` to shut down all libraries, update files in the application data directory with new ones, then restart the libraries. This provides a live update that never closes the Omnis runtime.

The flow would be:
 1. Download the updated file or files
 1. Copy the updated file or files to `firstruninstall/` in the program directory (this may require administrative rights)
 1. Message `app_loader.lbs` to reload the libraries using this call:

```omnis
Do $itasks.app_loader.$reloadLibrary($clib)
```

The reload message will trigger `app_loader.lbs` to:
 1. Shutdown the library triggering the reload
 1. Close all open libraries besides `app_loader.lbs`
 1. Update the libraries in the application data folder from `firstruninstall/`
 1. Start all libraries in `lib/`
 1. Start all libraries in `startup/`


## Contributing
Please see our [guide to contributing](https://github.com/suransys/contributing).