
Sieving-WC
==========

Web-Component for Sieving

<p/>
<img src="https://nodei.co/npm/sieving-wc.png?downloads=true&stars=true" alt=""/>

<p/>
<img src="https://david-dm.org/rse/sieving-wc.png" alt=""/>

About
-----

Sieving-WC is a Web Component for the Sieving JavaScript library.
It provides an HTML Element compatible with the regular &lt;textarea&gt;
but with syntax highlighting and tag completion.

Installation
------------

```shell
$ npm install sieving-wc
```

Usage
-----

```html
<!DOCTYPE html>
<html>
    <head>
        <script type="text/javascript" src="node_modules/sieving-wc/lib/sieving-wc.min.js"></script>
        <link rel="stylesheet" type="text/css" href="node_modules/sieving-wc/sieving-wc.css">
        [...]
    </head>
    <body class="sieving-wc-theme-light sieving-wc-theme-boxed">
        [...]
        <sieving-wc
            id="sieving"
            query="foo +bar -quux, baz"
        >
            <script type="application/json">
                { "#": [ "foo", "bar", "quux" ] }
            </script>
        </sieving-wc>
        <script type="text/javascript">
            let sieving = document.getElementById("sieving")
            sieving.addEventListener("change", (ev) => {
                let [ { query } ] = ev.detail
                console.log("Query:", query)
            })
        </script>
        [...]
    </body>
</html>
```

License
-------

Copyright &copy; 2020 Dr. Ralf S. Engelschall (http://engelschall.com/)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

