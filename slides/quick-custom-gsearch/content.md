## Whoami

<img class="avatar" data-src="https://avatars3.githubusercontent.com/u/6156742?v=3&s=460">

Kotaro Yoshimatsu

@ktrysmt

<https://twitter.com/ktrysmt>
<https://github.com/ktrysmt>

* 学生時代は Perl monger
* 受託開発会社 5年くらい
* リクルート 2年目

---

## What's this

- [Quick Custom GSearch \- Chrome ウェブストア](https://chrome.google.com/webstore/detail/quick-custom-gsearch/dcdmfmmmmpjgfaffnaokjpifnihmhaon)

You can access the control panel to manage interval quickly, just one click.

---

## Screenshot

<img style="width:80%" class="capture" src="https://raw.githubusercontent.com/ktrysmt/quick-custom-gsearch/master/capture.png">

---

## Why now develop chrome extensions

---

## Stand on the shoulders of chrome

- You can limit the scope to chrome.
- You can use stable and fulfilling Javascript ecosystem.
- Advanced technology APIs are available.
- No need Babel.

---

## How did I implement it

---

## Omitting basic practice ...

Ref:

- [Getting Started: Building a Chrome Extension \- Google Chrome](https://developer.chrome.com/extensions/getstarted)

Topics:

- Understand manifest.json
- Proprietary API in Google Chrome
- Option mechanism (if necessary)

---

dir tree (simplified)

```
.eslintrc
dist/
package.json
readme.md
src/
├── main.js
├── model.js
├── parser.js
└── view.js
test/
├── model.spec.js
├── parser.spec.js
├── setup.js
└── view.spec.js
webpack.config.js
yarn.lock
```

---

package.json

```json
{
  "devDependencies": {
    "chai": "^3.5.0",
    "eslint": "^3.13.0",
    "eslint-config-airbnb-base": "^11.0.0",
    "eslint-plugin-import": "^2.2.0",
    "jsdom": "^9.9.1",
    "mocha": "^3.2.0",
    "uglify-js": "git://github.com/mishoo/UglifyJS2#harmony",
    "webpack": "beta"
  },
  "scripts": {
    "start": "webpack -d -w",
    "lint": "eslint --fix src/*.js",
    "test": "mocha --require test/setup.js test/*.spec.js",
    "build:dev": "webpack -d",
    "build:prod": "webpack"
  }
}
```

---

### Yet Another

*Cannot apply stable-UglifyJS to ES6 as is.*

- Use Browserify or Babili. :sleeping:

*More faster test runner.*
*Want to test about async/await.*

- Use Ava. :triumph:
- Supports async/await and can execute test files in parallel.

---

.eslintrc

```json
{
  "extends": "airbnb-base",
  "env": { "browser": true },
  "rules": {
    "no-restricted-syntax": "off",
    "no-plusplus": "off",
    "no-unused-expressions": "off",
  }
}
```

---

webpack.config.js

```js
const webpack = require('webpack');
const path = require('path');

module.exports = {
  entry: {
    script: './src/main.js',
  },
  output: {
    path: path.resolve('dist/'),
    filename: '[name].js',
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new webpack.optimize.OccurrenceOrderPlugin(),
  ],
};
```

---

test/setup.js

```js
const jsdom = require('jsdom').jsdom;

const html = `<html><head></head><body>
<div id='ucs'><div id='main'></div></div>
</body></html>`;

global.document = jsdom(html, {
  url: 'https://www.google.co.jp/search?hl=ja&site=webhp&biw=810&bih=1306&q=duckduckgo&oq=duckduckgo&ie=UTF-8&tbs=qdr:y&tbm=',
});

global.window = document.defaultView;
Object.keys(document.defaultView).forEach((property) => {
  if (typeof global[property] === 'undefined') {
    global[property] = document.defaultView[property];
  }
});

global.navigator = {
  userAgent: 'node.js',
};
```

- referred to airbnb/enzyme


---


test/view.spec.js

```js
/* eslint-env mocha */
const expect = require('chai').expect;
const View = require('../src/view.js');
const Model = require('../src/model.js');

describe('View >', () => {
  //...

  it('CreateElement', () => {
    //...
  });

  it('BindElement', () => {
    //...
  });

  it('SetCssState', () => {
    //...
  });

  it('QuickChange', () => {
    //...
  });
});
```

--

```js
it('CreateElement', () => {
  const dom = View.CreateElement();
  const nodes = dom.childNodes;

  expect(nodes[0].nodeName).to.equal('SPAN');
  expect(nodes[7].nodeName).to.equal('SPAN');

  for (let i = 0; i < Model.TERM.length; ++i) {
    const key = i + 1;
    expect(nodes[key].nodeName).to.equal('A');
    expect(nodes[key].textContent).to.equal(Model.TERM[i].text);
    expect(nodes[key].getAttribute('data')).to.equal(Model.TERM[i].data);
  }
});
```

```js
it('BindElement', () => {
  const dom = View.CreateElement();
  View.BindElement(dom);
  const target = document.querySelector('div#ucs > .quick-custom-gsearch');
  expect(target.nodeName).to.equal('DIV');
});
```

---


## Other useful tools

[Chrome拡張の開発時にソースの更新を自動で反映させる \- Qiita](http://qiita.com/daisy1754/items/bbba1b0ad5f60c08706c)

- Contents scriptの開発に便利。

[github.com/acvetkov/sinon-chrome](https://github.com/acvetkov/sinon-chrome)

- GoogleChrome専用APIのSinonライブラリ

---

## conclusion

- Chrome first is justice.
- We can try modern Javascript practice and ecosystem right away.

---

## References

- [github.com/ktrysmt/quick-custom-gsearch](https://github.com/ktrysmt/quick-custom-gsearch)
- [github.com/acvetkov/sinon-chrome](https://github.com/acvetkov/sinon-chrome)
- [github.com/avajs/ava](https://github.com/avajs/ava)
- [enzyme/jsdom\.md at master · airbnb/enzyme](https://github.com/airbnb/enzyme/blob/master/docs/guides/jsdom.md)
- [Chrome拡張の開発時にソースの更新を自動で反映させる \- Qiita](http://qiita.com/daisy1754/items/bbba1b0ad5f60c08706c)
- [ググったあとワンクリックで期間指定ができるChrome拡張を作った \- Qiita](http://qiita.com/ktrysmt/items/87370a3ef4b5234e6e09)
