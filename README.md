# elm-performance-debugging
Simple way how to improve experience from performance profiling and debugging in Elm output code.

Usually during performance profiling you god list of many anonymous function calls and you need to manually go and click through them. 
But with simple regex aplied on compiled output, you can ged more human friendly names.

```JavaScript
/* replace FX nested anonymous functions with simple nesting
  Example:

  var $elm$core$List$filter = F2(
      function (isGood, list) {
        //some logic here
      }
  );

  Anonymous function will be named by
  __$elm$core$List$filter
*/

const rule = /var ([^=]+)( = F\d\([^f]+function)[^(]\(/gmi;
const replacemend = `var $1$2 __$1( `;
```

And another aditional expression:
```JavaScript
/* AX functions where FX is called like
  Example:
  return A3(
      $elm$core$List$foldr,
      F2(
        function THIS_WILL_BE_REPLACED (x, xs) {
          return isGood(x) ? A2($elm$core$List$cons, x, xs) : xs;
        }),
      _List_Nil,
      list);
  And THIS_WILL_BE_REPLACED will be replaced by `___$elm$core$List$foldr`
*/
consrt rule = /(\sA\d\([\s]+)([^,]+)(,[\s]+)(function[^(]+)\(/gm;
const replacemend = `$1$2$3$4___$2(`;
```

Example how this can be used with webpack. This code is using `string-replace-loader` webpack loader. 

```JavaScript
{
  test: /\.elm$/,
  exclude: [/elm-stuff/, /node_modules/],
  use: [
    // this crazy magic is only for localhost development
    devMode && perfDebug ? {
      loader: 'string-replace-loader',
      options: {
        multiple: [
          { /* replace FX nested anonymous functions with simple nesting
              Example:

              var $elm$core$List$filter = F2(
                  function (isGood, list) {
                    //some logic here
                  }
              );

              Anonymous function will be named by
              __$elm$core$List$filter
            */
            search:  /var ([^=]+)( = F\d\([^f]+function)[^(]\(/gmi,
            replace: `var $1$2 __$1( `,
          },
          { /* AX functions where FX is called like
              Example:
              return A3(
                  $elm$core$List$foldr,
                  F2(
                    function THIS_WILL_BE_REPLACED (x, xs) {
                      return isGood(x) ? A2($elm$core$List$cons, x, xs) : xs;
                    }),
                  _List_Nil,
                  list);
              And THIS_WILL_BE_REPLACED will be replaced by `___$elm$core$List$foldr`
            */
            search: /(\sA\d\([\s]+)([^,]+)(,[\s]+)(function[^(]+)\(/gm,
            replace: `$1$2$3$4___$2(`,
          }
        ]
      }
    } : null,
    devMode ? 'elm-hot-webpack-loader' : null,
    {
      loader: 'elm-webpack-loader',
      options: {
        optimize: !devMode,
        pathToElm: 'node_modules/.bin/elm'
      },
    },
  ].filter(x => x), // non-nulls
  }
}
```

And here example of performance recording in Chrome dev tools. 
![Performance recording example](/usage-example.png)
