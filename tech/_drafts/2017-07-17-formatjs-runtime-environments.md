---
layout: post
title: FormatJS指南 2 - Runtime Environments
tags: 原创 技术 翻译 React
---

[原文](https://formatjs.io/guides/runtime-environments/)

    <p>
        Here is information on how to setup {{brand}} in different environments, including how to polyfill the runtimes that don't have built-in internationalization support.
    </p>

    <section id="toc"></section>

    <section class="secs">
        <h2 id="intl-built-ins">
            {{code "Intl"}} Built-in APIs
        </h2>

        <p>
            The {{brand}} libraries rely on the following <a href="http://www.ecma-international.org/ecma-402/1.0/index.html">ECMA 402</a> Internationalization APIs introduced in ECMAScript 5.1:
        </p>

        <ul>
            <li>
                <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat">{{code "Intl.NumberFormat"}}</a>
            </li>
            <li>
                <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat">{{code "Intl.DateTimeFormat"}}</a>
            </li>
        </ul>

        <p>
            If these APIs aren't present in a runtime your app is targeting, then you'll need to polyfill the runtime to include them.
        </p>

        <section class="secs">
            <h3 id="intl-runtimes">
                Runtimes with {{code "Intl"}} APIs
            </h3>

            <p>
                The {{code "Intl"}} APIs are currently available on Node.js 0.12 and all modern browsers <em>except</em> Safari. See the <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl#Browser_Compatibility">Mozilla Developer Network documentation</a> for an up-to-date list of capable browsers.
            </p>
        </section>

        <section class="secs">
            <h3 id="intl-polyfill">
                Intl.js Polyfill
            </h3>

            <p>
                The <a href="https://github.com/andyearnshaw/Intl.js">Intl.js polyfill</a>, developed by Andy Earnshaw, provides a polyfill for the {{code "Intl.NumberFormat"}} and {{code "Intl.DateTimeFormat"}} APIs &mdash; both of which are needed by the {{brand}} libraries. The polyfill also includes separate locale data files for each of the 700+ locales it supports.
            </p>

            <p class="note">
                <strong>Note</strong> that the {{code "Intl.js"}} polyfill does not have the {{code "Intl.Collator"}} API. The size of the locale data required to support the API is not practical to load in a client.
            </p>
        </section>
    </section>

    <section class="secs">
        <h2 id="server">
            Server (Node.js)
        </h2>

        <p>
            Node.js 0.12 has the {{code "Intl"}} APIs built-in, <b>but only includes the English locale data by default</b>. If your app needs to support more locales than English, you'll need to <a href="https://github.com/nodejs/node/wiki/Intl">build or configure Node with the extra locale data</a>, or polyfill the runtime.
        </p>

        <p>
            Node.js versions prior to 0.12 don't provide the {{code "Intl"}} APIs, so they require that the runtime is polyfilled.
        </p>

        <section class="secs">
            <h3 id="polyfill-node">
                How to Polyfill Node.js
            </h3>

            <p>
                Node.js <= 0.10 doesn't have the built-in <code>Intl</code> APIs (ECMA-402). Node.js 0.12 does, but the default build/distribution only supports English.
            </p>

            <p>
                The following code snippet uses the <a href="https://github.com/andyearnshaw/Intl.js">{{code "intl"}}</a> and <a href="https://github.com/yahoo/intl-locales-supported">{{code "intl-locales-supported"}}</a> npm packages which will help you polyfill the Node.js runtime when the {{code "Intl"}} APIs are missing, or if the built-in {{code "Intl"}} is missing locale data that's needed for your app:
            </p>

{{#code "js"}}
var areIntlLocalesSupported = require('intl-locales-supported');

var localesMyAppSupports = [
    /* list locales here */
];

if (global.Intl) {
    // Determine if the built-in `Intl` has the locale data we need.
    if (!areIntlLocalesSupported(localesMyAppSupports)) {
        // `Intl` exists, but it doesn't have the data we need, so load the
        // polyfill and replace the constructors with need with the polyfill's.
        var IntlPolyfill = require('intl');
        Intl.NumberFormat   = IntlPolyfill.NumberFormat;
        Intl.DateTimeFormat = IntlPolyfill.DateTimeFormat;
    }
} else {
    // No `Intl`, so use and load the polyfill.
    global.Intl = require('intl');
}
{{/code}}

        </section>

        <section class="secs">
            <h3 id="user-locale-server">
                Determining the User's Locale
            </h3>

            <p>
                When a request comes in you'll need to determine the locale for the response. Best practice is to provide the user an explicit way of choosing one of the locales your app supports (and persisting that choice in a user profile database).
            </p>

            <p>
                If you wish to programmatically infer the user's locale you can inspect the <a href="http://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Accept-Language">{{code "Accept-Language"}}</a> HTTP request header. (This is part of HTTP <a href="http://en.wikipedia.org/wiki/Content_negotiation">content negotiation</a>.) There are <a href="https://www.npmjs.org/search?q=accept-language">npm modules</a> that can help with this, {{npmLink "accepts"}} and {{npmLink "negotiator"}} being two good choices. As well, express has some content negotiation <a href="http://expressjs.com/4x/api.html#req.acceptsLanguages">built in</a>.
            </p>

{{#code "js"}}
var app        = require('express')();
var appLocales = ['de', 'en', 'fr'];

app.get('/', function (req, res) {
    var locale = req.acceptsLanguages(appLocales) || 'en';
    // ...customize the response to locale
});
{{/code}}

        </section>
    </section>

    <section class="secs">
        <h2 id="client">
            Client (Browsers)
        </h2>

        <p>
            The {{code "Intl"}} APIs are currently available on all modern browsers <em>except</em> Safari. See the <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl#Browser_Compatibility">Mozilla Developer Network documentation</a> for an up-to-date list of capable browsers.
        </p>

        <p>
            It is recommended that you separate the {{code "Intl"}} polyfill from your app's JavaScript code bundle so the polyfill is only loaded in the browsers which need it. Also, if you're using the Intl.js polyfill, you will need to load a separate file for each locale. In this case you can dynamically load a single locale file for the user's locale.
        </p>

        <section class="secs">
            <h3 id="polyfill-browsers">
                How to Polyfill Browsers
            </h3>

            <p>
                The <a href="https://github.com/andyearnshaw/Intl.js">Intl.js polyfill</a>, developed by Andy Earnshaw, provides a polyfill for the {{code "Intl"}} API. It needs a locale data file for the current page.
            </p>

{{#code "html"}}
<script src="path/to/intl/Intl.js"></script>
<script src="path/to/intl/locale-data/jsonp/en.js"></script>
...use the locale for the current user, instead of hard-coding to "en"
{{/code}}

            <p>
                Since some runtimes already have the {{code "Intl"}} APIs, ideally you should only load the polyfill when needed. You can do this using a conditional loader.
            </p>

            <p>
                One conditional loader is <a href="http://yepnopejs.com/">yepnopejs</a>. Here's an overview of how to use it and it provides the general idea for other loading mechanisms:
            </p>

{{#code "js"}}
yepnope({
    test    : window.Intl,
    nope    : ['path/to/intl/polyfill'],
    complete: function () {
    // ...the Intl API can be used here...
    }
});
{{/code}}

            <p class="note">
                <strong>Note</strong> that our <a href="{{pathTo 'integrations'}}">integrations</a> require that {{code "window.Intl"}} exists before they are loaded.
            </p>
        </section>

        <section class="secs">
            <h3 id="user-locale-client">
                Determining the User's Locale
            </h3>

            <p>
                <strong>When running internationalization code in the browser it is best if the locale used is the same as was used when the page was generated on the server.</strong> You can do this by having the server embed the chosen locale into the generate page. This ensures that the user gets a consistent experience and that the UI doesn't suddenly "switch languages" on them.
            </p>

            <p>
                If this isn't possible or if you have an application which is served statically, the best practice is to provide the user an explicit way to choose one of the locales your app supports. If you wish to programmatically infer the user's locale you can match the following against the locales your app supports.
            </p>

{{#code "js"}}
navigator.language || navigator.browserLanguage
{{/code}}

        </section>
    </section>
