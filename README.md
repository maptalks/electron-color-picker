# @maptalks/electron-color-picker

Fork of [https://github.com/mockingbot/electron-color-picker](https://github.com/mockingbot/electron-color-picker) to support DPI aware under Windows.

[![i:npm]][l:npm]
[![i:size]][l:size]
[![i:npm-dev]][l:npm]

Pick color from Desktop, in Electron.

[i:npm]: https://img.shields.io/npm/v/electron-color-picker?colorB=blue
[i:npm-dev]: https://img.shields.io/npm/v/electron-color-picker/dev
[l:npm]: https://npm.im/electron-color-picker
[i:size]: https://packagephobia.now.sh/badge?p=electron-color-picker
[l:size]: https://packagephobia.now.sh/result?p=electron-color-picker

[//]: # (NON_PACKAGE_CONTENT)


* [Example](#example)
* [Usage](#usage)
* [Advanced Usage](#advanced-usage)


> #### Note
> 
> On Windows & MacOS will use our native [color-picker](https://github.com/mockingbot/mb_colorpicker_desktop_native).
> 
> On Linux will use [SCROT][l:scrot] to get screenshot and pick color from it.
> The idea is directly borrowed from package [desktop-screenshot][l:desktop-screenshot].
> 
> Error will be thrown:
> - when try to start multiple color-picker.
> - on unsupported platform.

[l:scrot]: https://en.wikipedia.org/wiki/Scrot
[l:desktop-screenshot]: https://npm.im/desktop-screenshot


## Example

📁 [example/](example/)

Basic implementation of using DOM Button to trigger color picking,
and pass result back through `ipc`.

Try example with:
```bash
cd example/

npm install

npm run-dev # for debug with electron
# or
npm run-prod # for electron-packager output
```

For a more detailed explanation of the example setup,
check: [example/concept.md](example/concept.md)


## Usage

First add this package to your project: 
```bash
npm install @maptalks/electron-color-picker
```

Sample function `saveColorToClipboard()`:
```js
const { clipboard } = require('electron')
const {
  getColorHexRGB,

  // for more control and customized checks
  DARWIN_IS_PLATFORM_PRE_CATALINA, // darwin only, undefined on other platform
  darwinGetColorHexRGB, // darwin only, throw error on other platform
  darwinGetScreenPermissionGranted, // darwin only, throw error on other platform
  darwinRequestScreenPermissionPopup // darwin only, throw error on other platform
} = require('electron-color-picker')

const saveColorToClipboard = async () => {
  // color may be '#0099ff' or '' (pick cancelled with ESC)
  const color = await getColorHexRGB().catch((error) => {
    console.warn('[ERROR] getColor', error)
    return ''
  })
  console.log(`getColor: ${color}`)
  color && clipboard.writeText(color)
}

if (process.platform === 'darwin' && !DARWIN_IS_PLATFORM_PRE_CATALINA) {
  const darwinScreenPermissionSample = async () => {
    const isGranted = await darwinGetScreenPermissionGranted() // initial check
    console.log('darwinGetScreenPermissionGranted:', isGranted)
    if (!isGranted) { // request user for permission, no result, and will not wait for user click
      await darwinRequestScreenPermissionPopup()
      console.warn('no permission granted yet, try again')
      return ''
    }
    const color = await darwinGetColorHexRGB().catch((error) => {
      console.warn('[ERROR] getColor', error)
      return ''
    })
    console.log(`getColor: ${color}`)
    color && clipboard.writeText(color)
  }
}
```


## Advanced Usage

To pull deeper code for specific platform,
consider direct require/import the platform `index` file.
Do read the source code and check the functionality though.

```js
const { // code with simple mutex wrapper
  getColorHexRGB,
  DARWIN_IS_PLATFORM_PRE_CATALINA,
  darwinGetColorHexRGB,
  darwinGetScreenPermissionGranted,
  darwinRequestScreenPermissionPopup
} = require('@maptalks/electron-color-picker')

const { // platform specific function, with less check
  runColorPicker,
  DARWIN_IS_PLATFORM_PRE_CATALINA,
  darwinRunColorPicker,
  darwinGetScreenPermissionGranted,
  darwinRequestScreenPermissionPopup
} = require('@maptalks/electron-color-picker/library/darwin/index.js')
```
