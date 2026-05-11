# Development Optimization Tools

Build automation tools that streamline development workflows.

## Overview

Tools like **Gulp**, **Grunt**, **Vite**, and **webpack** are task runners / bundlers that automate build steps:

- Minifying JavaScript and CSS
- Compressing images
- Compiling Sass/LESS
- Running tests
- Watching files for changes
- Hot module replacement (HMR) during development

---

## Gulp

**Task-based build tool** with stream-based processing.

```javascript
// gulpfile.js
const gulp = require('gulp');
const uglify = require('gulp-uglify');
const sass = require('gulp-sass');

gulp.task('js', function() {
  return gulp.src('src/**/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('dist/'));
});

gulp.task('css', function() {
  return gulp.src('src/**/*.scss')
    .pipe(sass())
    .pipe(gulp.dest('dist/'));
});

gulp.task('default', gulp.series('js', 'css'));
```

Run with: `gulp`

---

## Grunt

**Configuration-based build tool.**

Similar to Gulp but typically more verbose config files.

```javascript
// Gruntfile.js
module.exports = function(grunt) {
  grunt.initConfig({
    uglify: {
      my_target: {
        files: {
          'dist/output.min.js': ['src/**/*.js']
        }
      }
    },
    sass: {
      dist: {
        files: {
          'dist/style.css': 'src/style.scss'
        }
      }
    }
  });

  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-sass');
  grunt.registerTask('default', ['uglify', 'sass']);
};
```

Run with: `grunt`

---

## Vite

**Modern, fast frontend build tool.** Significantly faster than webpack for development.

### Key Advantages

- **Lightning-fast cold starts** — Uses native ES modules in development
- **Instant HMR** — Hot module replacement is near-instantaneous
- **Optimized builds** — Uses esbuild for production builds
- **Out-of-the-box support** — Vue, React, Svelte, Vanilla JS

### Configuration

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true  // Open browser on start
  },
  build: {
    target: 'esnext',
    minify: 'terser'
  }
})
```

### Commands

```bash
# Development server with HMR
npm run dev

# Production build
npm run build

# Preview production build
npm run preview
```

---

## Webpack

**Powerful bundler** for complex projects.

### Overview

Webpack treats everything as a module: JavaScript, CSS, images, fonts.

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: './dist/'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader'
        }
      }
    ]
  }
};
```

### Loaders

Transform non-JavaScript assets:
- `babel-loader` — Transpile modern JavaScript
- `css-loader` — Process CSS imports
- `file-loader` — Handle images, fonts

### Plugins

Perform actions on bundled code:
- `HtmlWebpackPlugin` — Generate HTML file
- `MiniCssExtractPlugin` — Extract CSS to file
- `DefinePlugin` — Define environment variables

---

## Image Optimization

Tools like **imagemin** can losslessly compress images and generate multiple sizes for responsive images.

### Standalone Tool

```bash
npm install --save-dev imagemin imagemin-mozjpeg imagemin-pngquant

# Compress images
imagemin src/images/* --out-dir=dist/images
```

### Gulp Integration

```javascript
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');

gulp.task('images', function() {
  return gulp.src('src/images/*')
    .pipe(imagemin([
      imagemin.mozjpeg({quality: 75}),
      imagemin.pngquant()
    ]))
    .pipe(gulp.dest('dist/images'));
});
```

### Responsive Images

Generate multiple sizes:

```bash
imagemin src/photo.jpg --output-type=webp --output-dir=dist
```

Then use with `srcset`:

```html
<img 
  srcset="
    photo-320w.jpg 320w,
    photo-640w.jpg 640w,
    photo-1280w.jpg 1280w"
  src="photo-640w.jpg"
  alt="Description"
>
```

**Goal:** Serve the smallest appropriate file without perceptible quality loss.

---

## Modern Frontend Tooling

### Development Stacks

**React/Vite Stack:**
```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

**Vue/Vite Stack:**
```bash
npm create vite@latest my-app -- --template vue
```

### Package Managers

- **npm** — Default, comes with Node.js
- **yarn** — Faster, deterministic lockfiles
- **pnpm** — Space-efficient, monorepo support

### Build Best Practices

1. **Minify** JavaScript and CSS
2. **Compress** images (PNG, JPEG, WebP)
3. **Tree-shake** unused code
4. **Code split** for lazy loading
5. **Cache busting** with content hashes
6. **Source maps** for production debugging

---

## Personal Toolchain Notes

*Document your preferred tooling stack and configuration patterns here.*

**Currently Using:**
- Frontend: Vite (fast, modern)
- Bundler: esbuild (built into Vite)
- Package manager: npm
- Build optimization: Built into Vite

**Want to Incorporate:**
- Detailed Vite configuration for personal app template

---

## Next Steps

- [Vite Documentation](https://vitejs.dev)
- [Webpack Documentation](https://webpack.js.org/)
- [Image Optimization Best Practices](https://web.dev/image-optimization/)
