---
title: "[OpenSource] 초보 컨트리뷰터의 Mocha issue 해결 도전기"
date: "2020-09-06 18:19:00"
template: "post"
draft: false
category: "OpenSource"
tags:
  - "OpenSource"
  - "Mocha"
  - "Contribution"
description: "저는 현재 오픈소스 컨트리뷰톤 Mocha팀에서 멘티로 활동하고 있습니다. Mocha의 #4433 이슈를 해결하면서 배운 점들을 적어봤습니다."
---

저는 현재 오픈소스 컨트리뷰톤 Mocha팀에서 멘티로 활동하고 있습니다. Mocha의 #4433 이슈를 해결하면서 배운 점들을 적어봤습니다. 최대한 raw하게, 혼잣말 하듯이 의식의 흐름대로 작성해보았습니다. 양해부탁드립니다 :) 저처럼 오픈소스에 대한 경험이 많이 없고, 두려운 초보자들에게 큰 도움이 되었으면 합니다.

# Mocha does not pass "--prof" parameter to Node and thus does not allow for profiling (#4433)

[Mocha does not pass "--prof" parameter to Node and thus does not allow for profiling · Issue #4433 · mochajs/mocha](https://github.com/mochajs/mocha/issues/4433)

mocha에서 Node의 `—prof` argument를 넘겨줬는데 Node가 처리하지 못하는 이슈이다.

멘토님께서 `lib/cli/node-flags.js`를 참고하라고 해주셨다. 링크를 열어보면, node에서 기본적으로 제공하는 flag의 목록이 보인다.

[Node.js v14.9.0 Documentation](https://nodejs.org/dist/latest-v12.x/docs/api/process.html#process_process_allowednodeenvironmentflags)

```jsx
./lib/cli/node-flags.js

'use strict';

/**
 * Some settings and code related to Mocha's handling of Node.js/V8 flags.
 * @private
 * @module
 */

const nodeFlags = process.allowedNodeEnvironmentFlags;
const {isMochaFlag} = require('./run-option-metadata');
```

코드를 보면 `process.allowedNodeEnvironmentFlags`안에 node의 flag들이 들어있다. 그리고 `./lib/cli/run-option-metadata`안에 mocha flag들이 들어있다. `—-prof`는 이 두 곳에 속해있지 않았다.

밑으로 내려가보면 다음 코드를 볼 수 있다. `node`와 `v8` 플래그를 여기서 따로 처리하는가보다. 이 목록에 `—-prof`가 빠져 있으니 추가하면 되지 않을까? 라는 생각이 먼저 들었다. 내가 해야하는 일은 `--prof` 플래그를 Node가 처리하도록 잘 넘겨주는 일이기 때문이다.

`—-prof`는 `node --v8-options`을 실행하면 나오는 옵션이기 때문이다. `—es-staging`도 여기서 찾을 수 있었다.  command 창에서 보기 힘들면, 다음 글에서 찾아보면 쉽다. 

[The Node.js Runtime v8 options list](https://flaviocopes.com/node-runtime-v8-options/)

먼저, `isNodeFlag`가 어디서 쓰이는지 찾아보았다. 

```jsx
./lib/cli/node-flags.js

/**
 * Mocha has historical support for various `node` and V8 flags which might not
 * appear in `process.allowedNodeEnvironmentFlags`.
 * These include:
 *   - `--preserve-symlinks`
 *   - `--harmony-*`
 *   - `--gc-global`
 *   - `--trace-*`
 *   - `--es-staging`
 *   - `--use-strict`
 *   - `--v8-*` (but *not* `--v8-options`)
 * @summary Whether or not to pass a flag along to the `node` executable.
 * @param {string} flag - Flag to test
 * @param {boolean} [bareword=true] - If `false`, we expect `flag` to have one or two leading dashes.
 * @returns {boolean} If the flag is considered a "Node" flag.
 * @private
 */
exports.isNodeFlag = (flag, bareword = true) => {
  if (!bareword) {
    // check if the flag begins with dashes; if not, not a node flag.
    if (!/^--?/.test(flag)) {
      return false;
    }
    // strip the leading dashes to match against subsequent checks
    flag = flag.replace(/^--?/, '');
  }
  return (
    // check actual node flags from `process.allowedNodeEnvironmentFlags`,
    // then historical support for various V8 and non-`NODE_OPTIONS` flags
    // and also any V8 flags with `--v8-` prefix
    (!isMochaFlag(flag) && nodeFlags && nodeFlags.has(flag)) ||
    debugFlags.has(flag) ||
    /(?:preserve-symlinks(?:-main)?|harmony(?:[_-]|$)|(?:trace[_-].+$)|gc(?:[_-]global)?$|es[_-]staging$|use[_-]strict$|prof$|v8[_-](?!options).+?$)/.test(
      flag
    )
  );
};
```

다음 파일에서 `isNodeFlag`를 찾을 수 있었고, `isNodeFlag`가 true이면 `nodeArgs`에 붙여주는 형식이었다. 

```jsx
./lib/cli/options.js

/**
 * Wrapper around `yargs-parser` which applies our settings
 * @param {string|string[]} args - Arguments to parse
 * @param {Object} defaultValues - Default values of mocharc.json
 * @param  {...Object} configObjects - `configObjects` for yargs-parser
 * @private
 * @ignore
 */
const parse = (args = [], defaultValues = {}, ...configObjects) => {
  // save node-specific args for special handling.
  // 1. when these args have a "=" they should be considered to have values
  // 2. if they don't, they just boolean flags
  // 3. to avoid explicitly defining the set of them, we tell yargs-parser they
  //    are ALL boolean flags.
  // 4. we can then reapply the values after yargs-parser is done.
  const nodeArgs = (Array.isArray(args) ? args : args.split(' ')).reduce(
    (acc, arg) => {
      const pair = arg.split('=');
      let flag = pair[0];
			console.log('isNodeFlag : ', isNodeFlag(flag, false));
      if (isNodeFlag(flag, false)) {
        flag = flag.replace(/^--?/, '');
        return arg.includes('=')
          ? acc.concat([[flag, pair[1]]])
          : acc.concat([[flag, true]]);
      }
      return acc;
    },
    []
  );

  const result = yargsParser.detailed(args, {
    configuration,
    configObjects,
    default: defaultValues,
    coerce: coerceOpts,
    narg: nargOpts,
    alias: aliases,
    string: types.string,
    array: types.array,
    number: types.number,
    boolean: types.boolean.concat(nodeArgs.map(pair => pair[0]))
  });
  if (result.error) {
    console.error(ansi.red(`Error: ${result.error.message}`));
    process.exit(1);
  }

  // reapply "=" arg values from above
  nodeArgs.forEach(([key, value]) => {
    result.argv[key] = value;
  });

  return result.argv;
};
```

아무 테스트 파일이나 실행시켜서, `—-es-staging` 옵션을 줄 때와 `—-prof` 옵션을 줄 때 결과가 똑같으면 될 것이라고 생각했다.

```bash
./bin/mocha ./test/only/bdd.spec.js --es-staging

console.log('isNodeFlag : ', isNodeFlag(flag, false));
[ [ 'es-staging', true ] ]
```

이 목록에 `—-prof`가 빠져있으니 추가해준다. `—-prof는 mochaFlag, nodeFlag, debugFlag`가 아니므로 `네번째 or문`에서 처리된다.

```jsx
./lib/cli/node-flags.js

/**
 * Mocha has historical support for various `node` and V8 flags which might not
 * appear in `process.allowedNodeEnvironmentFlags`.
 * These include:
 *   - `--preserve-symlinks`
 *   - `--harmony-*`
 *   - `--gc-global`
 *   - `--trace-*`
 *   - `--es-staging`
 *   - `--use-strict`
 *   - '--prof'
 *   - `--v8-*` (but *not* `--v8-options`)
 * @summary Whether or not to pass a flag along to the `node` executable.
 * @param {string} flag - Flag to test
 * @param {boolean} [bareword=true] - If `false`, we expect `flag` to have one or two leading dashes.
 * @returns {boolean} If the flag is considered a "Node" flag.
 * @private
 */
exports.isNodeFlag = (flag, bareword = true) => {
  if (!bareword) {
    // check if the flag begins with dashes; if not, not a node flag.
    if (!/^--?/.test(flag)) {
      return false;
    }
    // strip the leading dashes to match against subsequent checks
    flag = flag.replace(/^--?/, '');
  }
  return (
    // check actual node flags from `process.allowedNodeEnvironmentFlags`,
    // then historical support for various V8 and non-`NODE_OPTIONS` flags
    // and also any V8 flags with `--v8-` prefix
    (!isMochaFlag(flag) && nodeFlags && nodeFlags.has(flag)) ||
    debugFlags.has(flag) ||
    /(?:preserve-symlinks(?:-main)?|harmony(?:[_-]|$)|(?:trace[_-].+$)|gc(?:[_-]global)?$|es[_-]staging$|use[_-]strict$|prof$|v8[_-](?!options).+?$)/.test(
      flag
    )
  );
};
```

실행결과 똑같은 결과가 나왔다.

```bash
./bin/mocha ./test/only/bdd.spec.js --prof

console.log('isNodeFlag : ', isNodeFlag(flag, false));
[ [ 'prof', true ] ]
```

동작을 잘하는지 더 확실히 하고자 찾아봤는데, profiling을 하면 log file이 아웃풋 결과로 나온다고 한다. 잘 나오고 있었다!

[Easy profiling for Node.js Applications | Node.js](https://nodejs.org/en/docs/guides/simple-profiling/)

> Since we ran our application using the --prof option, a tick file was generated in the same directory as your local run of the application. It should have the form isolate-0xnnnnnnnnnnnn-v8.log (where n is a digit).

<div style="text-align: center">
<img src="https://user-images.githubusercontent.com/32301380/92322688-ba8b1500-f06d-11ea-84dd-0a1cd4f0268b.png"><br/>
<span>profiling 아웃풋 파일들</span>
</div>

추가적으로, 멘토님께서 `./test/node-unit/cli/node-flags-spec.js` 파일에서 테스트도 작성하면 좋을 거 같다고 하셨다. 그래서 아래 테스트 구문까지 추가해서 pr을 작성했다 🙂

```bash
it('should return true for "prof" itself', function() {
  expect(isNodeFlag('prof'), 'to be true');
});
```

[Add support for node v8 option 'prof' parameter (#4433) by Donghoon759 · Pull Request #4439 · mochajs/mocha](https://github.com/mochajs/mocha/pull/4439)

처음에는 되게 막막했는데, 코드를 따라가다보니 어디를 수정해야 될지 알 수 있었고, node flag를 주면 node가 처리하도록 넘기는구나, node에 profiling하는 옵션도 있구나 하는 새로운 내용도 배울 수 있어 좋았다. 정말 하면 할수록 알아가는 게 많았고, mocha 코드를 수정하는 pr은 처음이라 자신감도 붙었다!
