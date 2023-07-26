# webpack打包优化
### webpack/optimization
#### runtimeChunk
#### moduleIds
#### splitChunks

### webpack/externals
  ```js
    externals: {
      lodash: {
        commonjs: 'lodash',
        commonjs2: 'lodash',
        amd: 'lodash',
        root: '_',
      }
    }

    externals: [/^library\/.+$/,]
  ```