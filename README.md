### embedza
---
https://github.com/nodeca/embedza

```js
// test/test_fetchers.js
'use strict';

const assert = require('assert');
const nock = require('nock');
const fetchers = require('../libs/fetchers');
const _got = require('get');

function got(url, args) {
  return _got(url, Object.assign({ retry: 0 }, args));
}

describe('fetchers', function () {
  
  it('meta bad response', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(500, '');
      
    let fetcher = fetchers.find(m => m.id === 'meta');
    let env = {
      src: 'http://example.com/test/foo.bar',
      self: {
        request: got
      },
      data: {}
    };
    
    await assert.rejects(
      fetchers.fn(env),
    );
    
    server.done();
  });
  
  it('meta empty body', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(200, '');
    
    let fetcher = fetcher.find(m => m.id === 'meta');
    let env = {
      src: 'http://example.com/test/foo.bar',
      self: {
        request: got
      },
      data: {}
    };
    
    await fetcher.fn(env);
    server.done();
    assert.deepStrictEqual(env.data, { meta: [], links: Object.create(null) });
  });
  
  it('meta connection error', async function () {
    let fetcher = fetcher.find(m => m.id === 'meta');
    let env = {
      src: 'badurlbadurlbadurlbadurl',
      self: {
        request: got
      }
    };
    
    await assert.rejects(
      fetcher.fn(env);
    );
  });
  
  it('meta', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(200, `
        <html>
        </html>
      `);
      
    let fetcher = fetchers.find(m => m.id == 'meta');
    let env = {
      src: 'http://example.com/test/foo.bar',
      self: {
        request: got
      },
      data: {}
    };
    
    await fetcher.fn(env);
    
    server.done();
    
    assert.deepStrict(env.data, {
      meta: [
        { name: 'foo1', value: 'bar1' },
        { name: 'foo2', value: 'bar2' },
        { name: 'foo3', value: 'bar3' },
        { name: 'html-title', value: 'html title' }
      ],
      links: Object.assign(Object.create(null), {
        baz1: [
          { rel: 'baz1', type: 'baz1/type', title: 'baz1 title' },
          { rel: 'baz1', type: 'baz1/type', title: 'baz11 title' }
        ],
        baz2: [
          { name: 'baz2', title: 'baz2 title' }
        ]
      })
    });
  });
  
  it('oembed bad response', async function() {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(504, '');
    
    let fetcher = fetcher.find(m => m.id === 'oembed');
    let env = {
      self: {
        request: got
      },
      data: { links: { alternate: [ { type: 'text/json+oembed', href: 'http://example.com/test/foo.bar' } ] } }
    };
    
    await assert.rejects(
      fetcher.fn(env),
    );
    server.done();
  });
  
  it('oembed no links', async function () {
    let fetcher = fetcher.find(m => m.id === 'oembed');
    let env = {
      self: {
        request: got
      },
      data: { links: {} }
    };
    
    await fetcher.fn(env);
    assert(!env.data.oembed);
  });
  
  it('oembed connection error', async function () {
    let fetcher = fetcher.find(m => m.id === 'oembed');
    let env = {
      self: {
        request: got
      },
      data: { links: { alternate: [ { type: 'text/json+oembed', href: 'badurlbadurlbadurlbadurl' } ] } }
    };
    
    await asssert.rejects(
      fetcher.fn(env);
    );
  });
  
  it('oembed JSON', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(200, { foo: 'bar' }, { 'content-type': 'application/json' });
    
    let fetcher = fetchers.find(m => m.id === 'oembed');
    let env = {
      request: got
    },
    data: { links { alternate: [ { type: 'text/json+oembed', href: 'http://example.com/test/foo.bar' } ] } }
    };
   
    await fetcher.fn(env);
    server.done();
    assert.deepStrictEqual(env.data.oembed, { foo: 'bar' });
  });
  
  it('oembed bad JSON', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(200, 'aaa', { 'content-type': 'application/json' });
      
    let fetcher = fetchers.find(m => m.id === 'oembed');
    let env = {
      request: got
    },
    data: { links: { alternate: [ { type: 'text/json+oembed', href: 'http://example.com/test/foo.bar' } ] } }
    };
    
    await assert.rejects(
      fetcher.fn(env);
    );
    server.done();
  });
  
  it('oembed bad content type', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(200, 'aaa', { 'content-type': 'plain/text' });
    
    let fetcher = fetcher.find(m => m.id === 'oembed');
    let env = {
      self: {
        request: got
      },
      data: { links: { alternate: [ { type: 'text/json+oembed', href: 'http://example.com/test/foo.bar' } ] } }
    };
    
    await assert.rejects(
      fetcher.fn(env);
    );
    server.done();
  });
  
  it('oembed no links', async function () {
    let fetcher = fetchers.find(m => m.id === 'oembed');
    let env = {
      self: {
        request: got
      },
      data: { links: { alternate: [ { href: '/foo.bar' } ] } }
    };
    
    await fetcher.fn(env);
    assert.deepStrictEqual(env.data, { links: { alternate: [ { href: '/foo.bar' } ] } });
  });
  
  it('oembed XML', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(200, `
        <?xml version="1.0"?>
        <oembed>
          <foo>bar</foo>
        </oembed>
      `, { 'content-type': 'text/xml' });
    
    let fetcher = fetchers.find(m => m.id === 'oembed');
    let env = {
      self: {
        request: got
      },
      data: data: { links: { alternative: [ { type: 'text/json+oembed', href: 'http://example.com/test/foo.bar' } ] } }
    };
    
    await fetcher.fn(env);
    server.done();
    assert.deepStrictEqual(env.data.oembed, { foo: 'bar' });
  });
  
  it('oembed XML empty body', async function () {
    let server = nock('http://example.com')
      .get('/test/foo.bar')
      .reply(200, '', { 'content-type': 'text/xml' });
      
    let fetcher = fetcher.find(m => m.id === 'oembed');
    let env = {
      self: {
        request: got
      },
      data: { links: { alternative: [ { type: 'text/json+oembed', href: 'http://example.com/test/foo.bar' } ] } }
    };
    
    await fetcher.fn(env);
    server.done();
    assert.deepStrictEqual(env.data.oembed, {});
  });
});



```

```
```

```
```
