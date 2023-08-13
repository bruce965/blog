---
layout: page
title: "Universal Encoder/Decoder"
date: 2023-08-13 10:19:56 +0200
categories: [Tools]
tags: [tools, base64]
#hidden: true
---

This web tool can convert a text string from any format to any other format.
No server connection required, all conversions are performed client-side.

{% raw %}
<style>
.encdec-highlight {
  padding: 0;
}
.encdec-code-container {
  width: 100%;
  padding: 0 !important;
}
.encdec-code-block {
  width: 100%;
  height: 200px; /* TODO: auto-resize */
  background: none;
  outline: 0;
  padding: .5rem 1.5rem 1rem;
  border: 0;
  resize: none;
}
.encdec-select-container {
  display: inline-block;
  border-radius: .5rem;
}
.encdec-select-control {
  background: var(--highlight-bg-color);
  color: var(--code-header-text-color);
  border: 0;
  font-size: .85rem;
  font-weight: 600;
  text-align: center;
  cursor: pointer;
}
.encdec-select-control:hover {
  color: black;
}
@media (prefers-color-scheme: dark) {
  html:not([data-mode]) .encdec-select-control:hover,
  html[data-mode="dark"] .encdec-select-control:hover {
    color: white;
  }
}
.encdec-error {
  margin: 0 1em 1em;
}
.encdec-error:not([data-error]) {
  display: none;
}
.encdec-error::after {
  content: attr(data-error);
}
</style>

<div class="language-plaintext highlighter-rouge">
  <div class="code-header">
    <span class="encdec-select-container">
      <i class="fas fa-code fa-fw small"></i><select id="encdec-select1" class="encdec-select-control"></select>
    </span>
    <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button>
  </div>
  <div class="highlight encdec-highlight">
    <!-- HACK: https://github.com/cotes2020/jekyll-theme-chirpy/blob/0d4103d47bc9cff93918bb09a2957737cc3c9fe0/_includes/refactor-content.html#L14 -->
    <code><table class="rouge-table">
      <tbody>
        <tr>
          <td class="rouge-gutter gl"></td>
          <!-- HACK: avoid white spaces when copying to clipboard. -->
          <td class="rouge-code encdec-code-container"><textarea id="encdec-code1" class="encdec-code-block" contenteditable autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false">Hello, World!</textarea></td>
        </tr>
      </tbody>
    </table></code>
    <blockquote id="encdec-error1" class="prompt-danger encdec-error"></blockquote>
  </div>
</div>

<div class="language-plaintext highlighter-rouge">
  <div class="code-header">
    <span class="encdec-select-container">
      <i class="fas fa-code fa-fw small"></i><select id="encdec-select2" class="encdec-select-control"></select>
    </span>
    <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button>
  </div>
  <div class="highlight encdec-highlight">
    <code><table class="rouge-table">
      <tbody>
        <tr>
          <td class="rouge-gutter gl"></td>
          <td class="rouge-code encdec-code-container"><textarea id="encdec-code2" class="encdec-code-block" contenteditable autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false">SGVsbG8sIFdvcmxkIQ==</textarea></td>
        </tr>
      </tbody>
    </table></code>
    <blockquote id="encdec-error2" class="prompt-danger encdec-error"></blockquote>
  </div>
</div>

<script>(() => {
  const str2bin = data => new TextEncoder().encode(data);
  const bin2str = data => new TextDecoder('utf-8').decode(data);

  /** @type {{ [format: string]: string }} */
  const formats = {
    'base64': "Base64",
    'hex': "HEX",
    'json': "JSON",
    'uri-component': "URI Component",
    'utf8': "UTF-8",
  };

  /** @type {{ [format: string]: (data: string) => Uint8Array }} */
  const decoders = {
    'base64': data => str2bin(atob(data)), /* TODO: don't use atob */
    'hex': data => {
      data = data.replace(/[^a-zA-Z0-9]/g, '');

      if (/[^a-fA-F0-9]/.test(data))
        throw new SyntaxError("String contains an invalid character.");

      if (data.length % 2)
        throw new SyntaxError("Invalid string length.");

      const bytes = new Uint8Array(data.length / 2);
      for (let i = 0; i < data.length - 1; i += 2)
          bytes[i / 2] = parseInt(data.substr(i, 2), 16);

      return bytes;
    },
    'json': data => str2bin(JSON.parse(`"${data}"`)),
    'uri-component': data => str2bin(decodeURIComponent(data)),
    'utf8': str2bin,
  };

  /** @type {{ [format: string]: (data: Uint8Array) => string }} */
  const encoders = {
    'base64': data => btoa(bin2str(data)), /* TODO: don't use btoa */
    'hex': data => [...data].map(b => b.toString(16).padStart(2, '0')).join(''),
    'json': data => JSON.stringify(bin2str(data)).replace(/^"|"$/g, ''),
    'uri-component': data => encodeURIComponent(bin2str(data)),
    'utf8': bin2str,
  };

  const NO_CONVERTER_AVAILABLE = "No converter available.";

  const select1 = document.getElementById('encdec-select1');
  const select2 = document.getElementById('encdec-select2');

  const code1 = document.getElementById('encdec-code1');
  const code2 = document.getElementById('encdec-code2');

  const error1 = document.getElementById('encdec-error1');
  const error2 = document.getElementById('encdec-error2');

  for (const select of [select1, select2]) {
    for (const format in formats) {
      const option = document.createElement('option');
      option.value = format;
      option.innerText = formats[format];
      select.appendChild(option);
    }
  }

  const urlSearch = new URLSearchParams(window.location.search);
  select1.value = urlSearch.get('from') || 'utf8';
  select2.value = urlSearch.get('to') || 'base64';

  if (urlSearch.has('value'))
    code1.value = urlSearch.get('value');

  let format1 = select1.value;
  select1.addEventListener('change', () => {
    convert(code1.value, format1, select1.value, code1, error2, error1, true);
    format1 = select1.value;

    convert(code1.value, format1, format2, code2, error1, error2, false);
    refresh(code1.value, format1, format2);
  });

  let format2 = select2.value;
  select2.addEventListener('change', () => {
    convert(code2.value, format2, select2.value, code2, error1, error2, true);
    format2 = select2.value;

    convert(code2.value, format2, format1, code1, error2, error1, false);
    refresh(code2.value, format2, format1);
  });

  code1.addEventListener('input', () => {
    convert(code1.value, format1, format2, code2, error1, error2, false);
    refresh(code1.value, format1, format2);
  });

  code2.addEventListener('input', () => {
    convert(code2.value, format2, format1, code1, error2, error1, false);
    refresh(code2.value, format2, format1);
  });

  const refresh = (value, from, to) => {
    /* HACK: for the "copy to clipboard" script. */
    code1.innerText = code1.value;
    code2.innerText = code2.value;

    const urlSearch = new URLSearchParams(window.location.search);
    urlSearch.set('from', from);
    urlSearch.set('to', to);

    if (value)
      urlSearch.set('value', value);
    else
      urlSearch.delete('value');

    const url = window.location.pathname + '?' + urlSearch.toString();
    if (window.location.href != url)
      history.replaceState(null, "", url);
  };

  /**
   * @param {string} data
   * @param {string} srcFormat
   * @param {string} destFormat
   * @param {HTMLPreElement} dest
   * @param {HTMLDivElement} srcError
   * @param {HTMLDivElement} destError
   * @param {boolean} ignoreErrors
   */
  const convert = (data, srcFormat, destFormat, dest, srcError, destError, ignoreErrors) => {
    srcError.removeAttribute('data-error');
    destError.removeAttribute('data-error');

    /** @type {Uint8Array} */
    let decoded = null;

    if (Object.prototype.hasOwnProperty.call(decoders, srcFormat)) {
      try {
        decoded = decoders[srcFormat](data);
      }
      catch (e) {
        if (!ignoreErrors)
          dest.value = "ERROR";

        srcError.setAttribute('data-error', String(e.message || e));
        return;
      }
    } else {
      if (!ignoreErrors)
        dest.value = "ERROR";

      srcError.setAttribute('data-error', NO_CONVERTER_AVAILABLE);
      return;
    }

    if (Object.prototype.hasOwnProperty.call(encoders, destFormat)) {
      try {
        dest.value = encoders[destFormat](decoded);
      }
      catch (e) {
        if (!ignoreErrors)
          dest.value = "ERROR";

        destError.setAttribute('data-error', String(e.message || e));
        return;
      }
    } else {
      if (!ignoreErrors)
        dest.value = "ERROR";

      destError.setAttribute('data-error', NO_CONVERTER_AVAILABLE);
      return;
    }
  };

  convert(code1.value, format1, format2, code2, error1, error2, false);
  refresh(code1.value, format1, format2);
})()</script>
{% endraw %}
