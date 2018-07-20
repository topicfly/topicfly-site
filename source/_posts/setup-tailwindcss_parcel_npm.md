---
title: Setup Tailwindcss with Parcel.js or npm.
date: 2018-07-20
author: Hector Yeomans
tags:
  - css
  - tailwind
  - parcel
  - npm
---

## Why Tailwindcss?

I still struggle with CSS. I know the theory, I've read it multiple times and in paper it makes sense. Once I'm there writing HTML and trying to style it with CSS it does not make any click in my mind.

Tailwindcss goes the opposite way from full component frameworks like Bootstrap or Material UI. Tailwind is described as an utility first framework. Instead of having a class of `btn btn-primary` it will give you the classes to make a button, anchor or div a button exactly as you need it with a collection of classes: `bg-blue hover:bg-blue-dark text-white font-bold py-2 px-4 rounded`.

<!-- more -->

I've worked in companies that have used a similar system to Tailwindcss and I know it works for a big development team. If you want proof of something that has been done with Tailwind, one of the main contributors for Tailwind has made excellent tutorials:

* [Rebuilding Netlify with Tailwind CSS
](https://youtu.be/_JhTaENzfZQ?t=139): Title should be, "Rebuilding __a screen__ from Netlify"
* [Rebuilding Coinbase with Tailwind CSS
](https://youtu.be/7gX_ApBeSpQ?t=502): Title should be, "Rebuilding Coinbase __dashboard__ with Tailwind CSS"
* [tailwindcomponents.com](tailwindcomponents.com): If you're looking for inspiration.

## How does it work?

Tailwindcss works like this:

* You install npm development dependency
* You need PostCSS to use tailwindcss, so you install postcss npm dependency.
* You generate Tailwind configuration file:
  * This is the file that will give you a base of all the possible classes that you can have.
  * This file dictates at build time, how many possible combinations you can have of colors, widths, heights, responsive utilities, etcetera.
  * You will ultimately modify this file to add your brand colors, add new spacing dimensions and possibly add/remove hover states for responsive.
* Once the file is generated, it's recommended that you trim down your css file because you will most likely won't use all possible combinations of css classes generated.
  * You can automate this with postprocessor utilities
  * The way they work is through parsing your html and seeing what classes are there and removing those unused classes from your final css file.
* Rarely you will need to create custom CSS, but one neat idea is to build components based in a [combination of your tailwindcss classes](https://tailwindcss.com/docs/extracting-components)

## What's the catch?

* The css file could become huge if you don't postprocess it. In development it's ok to have a 300 kb file, in production is not.
* At the beggining it will take time to define your components, but as time progresses you will have a clean css file that are truly components.
* You're not really using the cascading part of CSS. Some people like it, some people don't. I don't like the cascade, it's like inheritance in OOP without good constraints. Just pick your poison.

## How to set it up?

I will show you how to set it up with two tools: parcel and npm. I think Parcel is the easiest one and I might do a post about Webpack.

__Prerequisites__: 

* Node 
* npm 

### Parcel

First make sure that Parcel.js is installed globally:

```
> npm i -g parcel-bundler
```

Then you can [follow common configuration](#common-configuration) and come back to generate your files.

** HTML **

Because Parcel works by parsing all your files, you have to add the following line to `src/index.html` after `<title>`:

```html
<!-- ... -->
<title>Tailwindcss starter</title>
<!-- add the following line -->
<link rel="stylesheet" href="./tailwind.css">
<!-- ... -->
```

In your terminal type:

```
>open http://localhost:1234 && parcel src/index.html
```

Wait for the build in your terminal and the browser should auto-refresh when done. Now you can start modifying the HTML and continue your work.

### npm

[Follow common configuration](#common-configuration) and then come back to make some modifications.

Modify your `package.json` file to include a npm script to be able to build your css, you will set your source from `tailwind.css` and output to `src/style.css`:

```json
...
scripts: {
  "css": "postcss src/tailwind.css -o src/style.css"
}
...
```

** HTML ** 
Before generating all your files, add the following line after `<title>` in `src/index.html`:

```html
<!-- ... -->
<title>Tailwindcss starter</title>
<!-- add the following line -->
<link rel="stylesheet" href="style.css"> 
<!-- ... -->
```

As a last step, build your css and open `index.html`:

```
> npm run css && open src/index.html
```

Hopefully now you can start customizing your live-reload or `dist` folder.

### <a name="common-configuration"></a> Common configuration 

** Project configuration **

You can copy the following lines and paste them into your terminal, and explanation will come afterwards:

```
mkdir new_tailwind_project && cd $_ \
&& npm init -y \
&& npm install -E -D tailwindcss autoprefixer cssnano @fullhuman/postcss-purgecss \
&& touch postcss.config.js \
&& ./node_modules/.bin/tailwind init \
&& mkdir src && touch src/tailwind.css && touch src/index.html
```

Description by line:

1. Create new directory, change `new_tailwind_project` to whaterver name you want.
1. Create a `package.json` file
1. Add required development dependencies with explicit version numbers, for example tailwind will be: `0.6.4`
1. Create an empty postcss configuration file.
1. Generate tailwind configuration file.
1. Create `src` folder and the main CSS file where you can define your custom css.

** Postcss **

Modify your `postcss.config.js` like this:

```js
const tailwindcss = require('tailwindcss');
const purgecss = require('@fullhuman/postcss-purgecss');
const cssnano = require('cssnano');
const autoprefixer = require('autoprefixer');

module.exports = {
  plugins: [
    tailwindcss('./tailwind.js'),
    cssnano({
      preset: 'default',
    }),
    purgecss({
      content: ['./src/**/*.html'],
    }),
    autoprefixer,
  ],
};
```

[cssnano](https://cssnano.co/) and [postcss-purgecss](https://github.com/FullHuman/postcss-purgecss) will help you to keep your css file size under control. 

** Tailwind.css **

Then modify your `src/tailwind.css` file and add these three lines:

```
@tailwind preflight;
@tailwind components;
@tailwind utilities;
```


** HTML **

Add the following html to `src/index.html`:

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Tailwindcss starter</title>
  
</head>

<body class="font-sans font-hairline">
  <div class="container flex m-10">
    <div class="m-10">
      <div class="max-w-sm rounded overflow-hidden shadow-lg">
        <img src="https://images.unsplash.com/photo-1531928351158-2f736078e0a1?ixlib=rb-0.3.5&amp;ixid=eyJhcHBfaWQiOjEyMDd9&amp;s=670bf968a401450fa19ef565bac7e0fe&amp;dpr=1&amp;auto=format&amp;fit=crop&amp;w=1000&amp;q=80&amp;cs=tinysrgb"
          class="w-full">
        <div class="px-6 py-4">
          <div class="font-bold text-xl mb-2">The Coldest Sunset</div>
          <p class="text-grey-darker text-base">
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptatibus quia, nulla! Maiores et perferendis eaque, exercitationem
            praesentium nihil.
          </p>
        </div>
        <div class="px-6 py-4">
          <span class="inline-block bg-grey-lighter rounded-full px-3 py-1 text-sm font-semibold text-grey-darker mr-2">#photography</span>
          <span class="inline-block bg-grey-lighter rounded-full px-3 py-1 text-sm font-semibold text-grey-darker mr-2">#travel</span>
          <span class="inline-block bg-grey-lighter rounded-full px-3 py-1 text-sm font-semibold text-grey-darker">#winter</span>
        </div>
      </div>
    </div>

  </div>
</body>

</html>
```
