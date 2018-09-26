---
layout: post
title: "ReactJS: SCSS/SASS for apps created by create-react-app"
date: 2018-09-14 23:30:33 +0800
categories: react reactjs create-react-app scss sass
tags: [ react, reactjs, create-react-app, scss, sass ]
---

If you are working on a React project created using the create-react-app then
I'm pretty sure that you were also dissapointed to know that it didn't configure
the app to load scss. Here I'll show you how to configure your react app to load
SCSS/SASS files.

First we eject the webpack configurations.

{% highlight bash %}
yarn eject.
{% endhighlight %}
or
{% highlight bash %}
npm run eject
{% endhighlight %}

This will generate the *'scripts'* and *'config'* directories. The one we need
is on the *'config'* directory. Look for **config/webpack.config.dev.js** file.
This is the file we need to edit in order to configure the app to load the
SCSS/SASS files but before that we need to download and install the neccessary
loaders.

{% highlight bash %}
yarn add sass-loader node-sass extract-loader
{% endhighlight %}
or
{% highlight bash %}
npm install --save-dev sass-loader node-sass extract-loader
{% endhighlight %}


On the **config/webpack.config.dev.js**, find the line that contains
**test: /\.css$/,**  and modify it to look like **test: /\.(css|scss)$/,**
then add this line **require.resolve('sass-loader'),** to use the sass-loader.


Below is how the 'module' block on my **config/webpack.config.dev** looks like.

{% highlight javascript %}
module: {
  strictExportPresence: true,
  rules: [
    {
      test: /\.(js|jsx|mjs)$/,
      enforce: 'pre',
      use: [
        {
          options: {
            formatter: eslintFormatter,
            eslintPath: require.resolve('eslint'),
          },
          loader: require.resolve('eslint-loader'),
        },
      ],
      include: paths.appSrc,
    },
    {
      oneOf: [
        {
          test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
          loader: require.resolve('url-loader'),
          options: {
            limit: 10000,
            name: 'static/media/[name].[hash:8].[ext]',
          },
        },
        {
          test: /\.(js|jsx|mjs)$/,
          include: paths.appSrc,
          loader: require.resolve('babel-loader'),
          options: {
            cacheDirectory: true,
          },
        },
        {
          // test: /\.css$/,
          test: /\.(css|scss)$/,
          use: [
            require.resolve('style-loader'),
            {
              loader: require.resolve('css-loader'),
              options: {
                importLoaders: 1,
              },
            },
            {
              loader: require.resolve('postcss-loader'),
              options: {
                ident: 'postcss',
                plugins: () => [
                  require('postcss-flexbugs-fixes'),
                  autoprefixer({
                    browsers: [
                      '>1%',
                      'last 4 versions',
                      'Firefox ESR',
                      'not ie < 9', // React doesn't support IE8 anyway
                    ],
                    flexbox: 'no-2009',
                  }),
                ],
              },
            },
            require.resolve('sass-loader'),
          ],
        },
        {
          exclude: [/\.js$/, /\.html$/, /\.json$/],
          loader: require.resolve('file-loader'),
          options: {
            name: 'static/media/[name].[hash:8].[ext]',
          },
        },
      ],
    },
    // ** STOP ** Are you adding a new loader?
    // Make sure to add the new loader(s) before the "file" loader.
  ],
},
{% endhighlight %}

yarn start
or
npm start

