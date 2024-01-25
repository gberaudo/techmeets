# Transform your JS with recast

## Context

Typescript can only transpile your code if you omit the '.ts' extensions in your import code:

```ts
// src/titi.ts on the filesystem, but no .ts extension !!!
import Titi from './titi';
```

That is very painful because it produces *non* standard JS.

## Goal

Write a post-transpilation script to make that JS standard.

## Solution 1: fix_path.js

```js
function handleFile(filePath) {
  fs.readFile(filePath, 'utf-8', (error, content) => {
    const split = content.split('\n');
    let changed = false;
    const newSplit = split.map((line) => {
      if (!(line.startsWith('import') || line.startsWith('export')) || !line.endsWith(';') || line.endsWith(".js';") || line.includes("from 'cesium") || line === 'export default OLCesium;') {
        return line;
      }
      let newline = undefined;
      if (line.endsWith(".ts';")) {
        newline = line.replace(".ts';", ".js';");
      } else if (line.includes("from './") || line.includes("from '../")) {
        newline = line.replace("';", ".js';");
      }
      changed = true;
      return newline;
    });
    if (changed) {
      fs.writeFileSync(filePath, newSplit.join('\n'), {
        encoding: 'utf-8'
      });
    }
  });
}
```

Straightforward, hackish, but "works great"...
...
Well, what about *sourcemaps*?

## Solution 2: recast

Recast is a JS AST transformer. It parses your "JS" text files and gives you a representation of it that you can navigate and mutate. It is both very simple and very powerful; it is the base component of the jscodeshift tool by Facebook.
See https://github.com/benjamn/recast

```js
function sourceRewrite(inFile, source) {
  if (!source.startsWith('.') || source.endsWith('.js')) {
    return; // not a relative import, we  only deal with our code
  }
  const newSource = source + '.js';
  const newsourcePath = path.join(path.dirname(inFile), newSource);
  if (!fs.existsSync(newsourcePath)) {
    return; // we only point to existing js files
  }
  return newSource;
}

function rewriteJSEnds(inFile, ast) {
  const body = ast.program.body;
  for (const node of body) {
    switch (node.type) {
      case 'ImportDeclaration':
      case 'ExportNamedDeclaration': {
        if (!node.source) {
          continue;
        }
        const source = node.source.value;
        const newSource = sourceRewrite(inFile, source);
        if (newSource) {
          node.source.value = newSource;
        }
        break;
      }
      default: {
        // pass
      }
    }
  }
```

Well, and what about sourcemaps?

```js
    const inlineSourceMap = `
//# sourceMappingURL=data:application/json;base64,${btoa(JSON.stringify(output.map))}
`;
    fs.appendFileSync(p, inlineSourceMap);
```

## Conclusion

Recast is easy, very powerful and allows you to do things _right_ (remember the sourcemaps?).
I did that to publish a clean OL-Cesium package, the full script should appear soon on https://github.com/openlayers/ol-cesium/blob/master/buildtools/fix_paths_recast.js
