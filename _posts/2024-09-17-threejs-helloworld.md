---
title: How to quickly say `hello world` with `three.js`
date: 2024-09-17 10:15:00 +0700
categories: [Blogging, Tutorial]
tags: [threejs, nodejs, vitejs]
author: j12t
toc: true
comments: true
math: true
mermaid: false
media_subpath: /assets/img/posts/2024-09-17-threejs-helloworld
image:
    path: threejs.png
pin: false
---

<a target="_blank" href="https://threejs.org/">`three.js`</a>  is the most popular <a target="_blank" href="https://webglfundamentals.org/">WebGL</a> framework for front-end developers. This tutorial is created to guide you to quickly getting started with `Three.js`.

For further developments, consider reading <a target="_blank" href="https://threejs.org/docs/">`Three.js`'s official documentation</a> or taking the course at <a target="_blank" href="https://threejs-journey.com/">`three.js` journey</a>.

Here is the complete <i class="fa-brands fa-github"></i> <a target="_blank" href="https://github.com/jerapiblaze/threejs-helloworld">Source code</a> of this tutorial.

## Before you begin

### (Required) Install `NodeJS` and `npm`

You need to install <a target="_blank" href="https://nodejs.org/en"><i class="fa-brands fa-node-js"></i> NodeJS</a> first.

Windows 11:
```console
$ winget install OpenJS.NodeJS.LTS
```

MacOS:
```console
$ brew install node@20
```

Linux/Ubuntu:
```console
$ sudo apt install nodejs
```

Verify that `NodeJS` and `npm` has installed successfully:
```console
$ node --version
v22.8.0
$ npm -v
10.8.2
```

### (Optional) Install `git`, `VisualStudio Code`

Windows 11:
```console
$ winget install Microsoft.VisualStudioCode Git.Git
```

MacOS:
```console
$ brew install git
$ brew install --cask visual-studio-code
```

Linux/Ubuntu:
```console
$ sudo apt update
$ sudo apt install software-properties-common apt-transport-https wget
$ sudo apt install git
$ wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
$ sudo apt install code
```

Now, you should have `Git` and `Visual Studio Code` installed. To enable support for `NodeJS` projects in `vscode`, run the following:

```console
code --install-extension 1YiB.nodejs-bundle
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> On Windows, you need to close the current and start a new terminal window to be able to use the command above.
{: .prompt-tip }
<!-- markdownlint-restore -->

### (Required) Terminals

You should know the basics about terminals, like `mkdir`, `ls`, and `cd` command.

## Start a new project

### Create and initialize new project folder

<b>1.</b> Open your terminal and execute the following commands:

```console
$ mkdir threejs-helloworld
$ cd threejs-helloworld
$ npm init
```

<b>2.</b> Press `enter` to accept all the prompts. Your terminal should look like this:

![npm init command result](npminit-result.png)

### Configuring packages

<b>3.</b> Now, install neccessary libraries using `npm install` command:

```console
$ npm install vite three
```

The results may look like below:

![npm install result](npminstall-result.png)

### First `threejs` webpage

<b>4.</b> Open `vscode` by the command:
```console
code .
```

![Opening vscode in project folder](vscode-open.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> You can open `explorer` right pane by <kbd>View</kbd> > <kbd>Appearance</kbd> > <kbd> Primary Side Bar </kbd> or using the keyboard shortcut  <kbd>ctrl</kbd>+<kbd>b</kbd>.
{: .prompt-tip }
<!-- markdownlint-restore -->

<b>5.</b> Now, using `Visual Studio Code`, open `package.json` and modify the `"scripts"` section as follow:

```js
{
  // ...
  // no changes
  "scripts": {
    "dev": "vite serve src",
    "build": "vite build src --outDir ../dist --emptyOutDir",
    "preview": "vite preview"
  },
  // no chages
  // ...
}

```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Don't edit any other sections !!!
{: .prompt-warning }
<!-- markdownlint-restore -->

<b>6.</b> Then, create new folder `src`, then inside that create three files: `index.html`, `script.js` and `stylesheet.css`. The final result should look like below:

![Project Folder structure](folder-structure.png)

<b>7.</b> Open `index.html` and paste the following lines:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>First Three.js Project</title>
    <script type="module" src="./script.js"></script>
    <link rel="stylesheet" href="./stylesheet.css">
</head>
<body>
    <canvas class="webgl" style="position:absolute; left:0px; top:0px;"></canvas>
</body>
</html>
```

<b>8.</b> Open `stylesheet.css` and paste the following lines:

```css
html, body {
  width: 100%;
  height: 100%;
  margin: 0px;
  border: 0;
  overflow: hidden;
  /*  Disable scrollbars */
  display: block;
  /* No floating content on sides */
}
```

<b>9.</b> Open `script.js` and paste the following lines:

```js
import * as THREE from 'three'
// Canvas
const canvas = document.querySelector('canvas.webgl')

// Sizes = Window size
const sizes = {
    // width: 800,
    width: window.innerWidth,
    // height: 600
    height: window.innerHeight
}

// Scene
const scene = new THREE.Scene()

// Objects
const geometry = new THREE.BoxGeometry(1, 1, 1)
const material = new THREE.MeshBasicMaterial({ color: 0xff0000 })
const mesh = new THREE.Mesh(geometry, material)
mesh.position.x = 0
mesh.position.y = 0
mesh.position.z = 0

const geometry2 = new THREE.SphereGeometry(1,128,256)
const material2 = new THREE.MeshBasicMaterial({color: 0xffffff})
const mesh2 = new THREE.Mesh(geometry2, material2)
mesh2.position.x = 3
mesh2.position.y = 2
mesh2.position.z = 0

// Put objects to scene
scene.add(mesh)
scene.add(mesh2)

// Camera
const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height)
camera.position.z = 5
camera.position.y = 2
camera.position.x = 1
scene.add(camera)

// Renderer
const renderer = new THREE.WebGLRenderer({
    canvas: canvas
})
renderer.setSize(sizes.width, sizes.height)

// Execute the renderer
renderer.render(scene, camera)
```

<b>10.</b> Now, run the command using the terminal:

```console
$ npm run dev
```

The command should returns as follow:

![npm run dev result](npmrundev-result.png)

Then head to the location in `local` line, in my case `http://localhost:5173/` to see the results. The result should look like this:

![Final result](final-result.png)

Congratulation! You have just said `helloworld` using `three.js`.

## Breakdown the files

### HTML file `index.html`

First, take a look at `index.html` file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>First Three.js Project</title>
    <script type="module" src="./script.js"></script>
    <link rel="stylesheet" href="./stylesheet.css">
</head>
<body>
    <canvas class="webgl" style="position:absolute; left:0px; top:0px;"></canvas>
</body>
</html>
```

Here, we set the page's title as `First Three.js Project`, include the stylesheet `stylesheet.css` file and our main logic in `script.js` file. In `body`, we have only one element is `canvas`, which is then used by `three.js` to render. This `canvas` is fitted into the size of the current browser window.

### Script file `script.js`

Now, take a look at the `script.js` file.

```js
import * as THREE from 'three'
```

This line is used to import the `three.js` library.

```js
// Canvas
const canvas = document.querySelector('canvas.webgl')

// Sizes = Window size
const sizes = {
    // width: 800,
    width: window.innerWidth,
    // height: 600
    height: window.innerHeight
}
```

In these lines, we select the `canvas` element and define the size for the rendering logic equals to the size of the current browser window.

```js
// Scene
const scene = new THREE.Scene()

// Objects
const geometry = new THREE.BoxGeometry(1, 1, 1)
const material = new THREE.MeshBasicMaterial({ color: 0xff0000 })
const mesh = new THREE.Mesh(geometry, material)
mesh.position.x = 0
mesh.position.y = 0
mesh.position.z = 0

const geometry2 = new THREE.SphereGeometry(1,128,256)
const material2 = new THREE.MeshBasicMaterial({color: 0xffffff})
const mesh2 = new THREE.Mesh(geometry2, material2)
mesh2.position.x = 3
mesh2.position.y = 2
mesh2.position.z = 0

// Put objects to scene
scene.add(mesh)
scene.add(mesh2)
```

Then we create the scene with two object are one $1 \times 1$ cube at position $(0,0,0)$, colored in `0xff0000` (red) and a sphrere with radius $r=1$ at position $(3,2,0)$, colored in `0xffffff` (white).

```js
// Camera
const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height)
camera.position.z = 5
camera.position.y = 2
camera.position.x = 1
scene.add(camera)
```

After that, we setup the camera at position $(1,2,5)$ with $75^{\circ}$ field of view and aspect ratio equal to the aspect ratio of the current window size.

```js
// Renderer
const renderer = new THREE.WebGLRenderer({
    canvas: canvas
})
renderer.setSize(sizes.width, sizes.height)

// Execute the renderer
renderer.render(scene, camera)
```

Finnally, we create a renderer, tell it to use the `canvas` HTML element as destination and execute the render.
