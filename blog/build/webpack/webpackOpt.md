### webpack/optimization
#### runtimeChunk
#### moduleIds
#### splitChunks

### webpack/externals
  ```
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