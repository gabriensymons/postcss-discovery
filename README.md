# Discovery Results Comparing PostCSS to Sass

## TLDR;
**`sass-loader`** compiles faster than **`postcss-loader`** using Fenrir's Webpack build

## Discovery Ticket
[EC-4845](https://businessinsider.atlassian.net/browse/EC-4845) Investigate if PostCSS would be a valid replacement for Sass

## Branch
[`postcss-experiment`](https://github.com/businessinsider/fenrir/compare/postcss-experiment#diff)

The most notable configuration changes are in these files:
- [`package.json`](https://github.com/businessinsider/fenrir/compare/postcss-experiment#diff-7ae45ad102eab3b6d7e7896acd08c427a9b25b346470d7bc6507b6481575d519)
- [`webpack.config.js`](https://github.com/businessinsider/fenrir/compare/postcss-experiment#diff-1fb26bc12ac780c7ad7325730ed09fc4c2c3d757c276c3dacc44bfe20faf166f)

The other files are only used for testing purposes, which you can ignore.

## Problem
Fenrir's local build time increases each time we:
1. Add a new CSS segment
1. Add a new styled component
1. Add a new layout, module, or theme style sheet

## Hypothesis
It is possible to improve Fenrir’s local build time using PostCSS instead of SASS with:
- No changes to our CSS workflow
- No changes to our import structure
- Minimal changes to our style syntax

## Goals
- Investigate the level of effort to convert Fenrir from Sass to PostCSS
- Discover if there are improvements to our build times and bundle sizes

## Experiment Plan
1. Create a side by side comparison of PostCSS and Sass
1. Compare build times using a single CSS segment
1. Compare build times using several CSS segments with an import structure similar to Fenrir's current structure

## Result Summary
### Build Comparison
Build comple time with 1 module and 10 modules

 **Loader** | **1 module** | **10 modules**
:---|:---:|:---:
 **postcss-loader** | 2.62 secs | 2.51 secs 
 **sass-loader** | 1.27 secs | 1.47 secs 

### File Change Comparison
File change compile time with 1 module and 10 modules

 **Loader** | **1 module** | **10 modules** 
:---|:---:|:---:
 **postcss-loader** | 0.4 secs | 0.486 secs
 **sass-loader** | 0.321 secs | 0.374 secs
 
## Result Details
These reports compare the build time of `postcss-loader` to `sass-loader` for 1 module, and 10 modules.
They also compare the compile time after editing one file in a build with 1 module, and a build with 10 modules.

<img width="765" alt="Build Comparison" src="https://user-images.githubusercontent.com/493546/189436094-3734169d-4a34-44c9-826a-606dd90c3283.png">
<img width="764" alt="File Edit Comparison" src="https://user-images.githubusercontent.com/493546/189436104-2aa04515-71b6-4754-b1b7-a62abfd1e6bf.png">

 These reports output from a tool on Fenrir called [speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin). It measures the speed of  local webpack builds and reports how long each loader takes to run. You can output these reports by running `npm run dev:debug`.

## Notes
Turning off source maps slightly improves PostCSS build times, however it still compiles more slowly than SASS.

The `.pcss` file type is simply a `.css` file with syntax highlighting for SCSS syntax like `$`, `@import`, etc.

[`styleLoader.js`](https://github.com/businessinsider/fenrir/compare/postcss-experiment#diff-20b6136c7d946ec3d146b80ffd356c80b7b6735c00ac1a58c456889ceb3976a3) loads every component style as another module. So adding a component with style affects build time.

## Discovery Outcome
### Recommendation
Based on the outcome of comparing performance of Sass and PostCSS build times, I **recommend we continue using Sass** for the time being.

### Level of effort
I estimate that using PostCSS instead of SCSS would require the following level of effort:

#### Low
- Update `.scss` files to use PostCSS
- Setup stylelint for new `.pcss` files

#### Low to Medium
- Update Sass functions and mixin syntax
- Automatic live update of CSS on file change*
- Enable `@use` and other sass specific commands*

#### Medium to High
- Review each component style file*
- Get PostCSS working for `npm run build`*

*_Not addressed in this experiment_


### Plugins & Bundle Sizes

Name | Description | Minified | Gzipped
:---|:---|---:|---:
 [postcss](https://github.com/postcss/postcss) | Tool to transform CSS with JavaScript | 48.1kB | 14.8kB
 [postcss-cli](https://github.com/postcss/postcss-cli) | CLI for PostCSS | 22.5kB | 6.92kB
 [postcss-loader](https://github.com/postcss/postcss-loader) | PostCSS loader for Webpack | 178.4kB | 57.5kB
 [postcss-import](https://github.com/postcss/postcss-import#postcss-import) | PostCSS plugin to transform `@import` rules by inlining content | 35.5kB | 9.6kB
 [postcss-preset-env](https://github.com/csstools/postcss-plugins/tree/main/plugin-packs/postcss-preset-env) | PostCSS plugin to allows the use modern CSS today | 1.2MB | 165kB
 [postcss-simple-vars](https://github.com/postcss/postcss-simple-vars) | PostCSS plugin for `$` Sass-like variables | 2.4kB | 1.1kB
 [postcss-nested](https://github.com/postcss/postcss-nested) | PostCSS plugin to use `&` the way Sass does | 52.3kB | 13.3kB
 [postcss-scss](https://github.com/postcss/postcss-scss) | SCSS parser for PostCSS to use `//` inline comments | 6.8kB | 2.7kB
 [postcss-mixins](https://github.com/postcss/postcss-mixins) | PostCSS plugin for mixins | 92.2kB | 28.9kB


## Next Steps
Perhaps there is opportunity make component styles loaded by `styleLoader` more efficient?



