# Tailwind CSS Angular Application

# How to add Tailwind to Angular application

# Install dependencies
  npm install tailwindcss -D

# Now we will install @angular-builders/custom-webpack for a custom webpack build step and postcss for building Tailwind.
npm install @angular-builders/custom-webpack postcss -D

# Create webpack.config.js
function patchPostCSS(webpackConfig, tailwindConfig, components = false) {
  if(!tailwindConfig){
    console.error('Missing tailwind config :', tailwindConfig);
    return;
  }
  const pluginName = "autoprefixer";
  for (const rule of webpackConfig.module.rules) {
    if (!(rule.use && rule.use.length > 0) || (!components && rule.exclude)) {
      continue;
    }
    for (const useLoader of rule.use) {
      if (!(useLoader.options && useLoader.options.postcssOptions)) {
        continue;
      }
      const originPostcssOptions = useLoader.options.postcssOptions;
      useLoader.options.postcssOptions = (loader) => {
        const _postcssOptions = originPostcssOptions(loader);
        const insertIndex = _postcssOptions.plugins.findIndex(
          ({ postcssPlugin }) => postcssPlugin && postcssPlugin.toLowerCase() === pluginName
        );
        if (insertIndex !== -1) {
           _postcssOptions.plugins.splice(insertIndex, 0, ["tailwindcss", tailwindConfig]);
        } else {
          console.error(`${pluginName} not found in postcss plugins`);
        }
        return _postcssOptions;
      };
    }
  }
}
module.exports = (config) => {
  const tailwindConfig = require("./tailwind.config.js");
  patchPostCSS(config, tailwindConfig, true);
  return config;
};

# Modify angular.json
ng config projects.<your-project>.architect.build.builder @angular-builders/custom-webpack:browser
ng config projects.<your-project>.architect.build.options.customWebpackConfig.path webpack.config.js
ng config projects.<your-project>.architect.serve.builder @angular-builders/custom-webpack:dev-server
ng config projects.<your-project>.architect.test.builder @angular-builders/custom-webpack:browser
ng config projects.<your-project>.architect.test.options.customWebpackConfig.path webpack.config.js

# Initialize TailwindCSS
npx tailwind init

# Inject Tailwind’s Styles
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';
