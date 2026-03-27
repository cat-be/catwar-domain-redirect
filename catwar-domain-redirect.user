// ==UserScript==
// @name         Перенаправление ссылок CatWar
// @namespace    https://catwar.su/
// @version      1.1
// @description  Заменяет ссылки с catwar.net на catwar.su с сохранением полного пути
// @author       1080554
// @match        *://catwar.net/*
// @match        *://catwar.su/*
// @grant        none
// @run-at       document-start
// @downloadURL  https://raw.githubusercontent.com/cat-be/catwar-domain-redirect/main/catwar-domain-redirect.user.js
// @updateURL    https://raw.githubusercontent.com/cat-be/catwar-domain-redirect/main/catwar-domain-redirect.user.js
// ==/UserScript==

(function () {
    'use strict';

    const FROM = 'https://catwar.net';
    const TO = 'https://catwar.su';

    function replaceDomain(url) {
        if (!url || typeof url !== 'string') return url;
        return url.startsWith(FROM) ? TO + url.slice(FROM.length) : url;
    }

    // Если пользователь открыл сам сайт catwar.net — сразу перенаправляем
    if (location.href.startsWith(FROM)) {
        location.replace(TO + location.href.slice(FROM.length));
        return;
    }

    function processElement(el) {
        if (!el || el.nodeType !== 1) return;

        // Обрабатываем href, src, poster
        ['href', 'src', 'poster'].forEach(attr => {
            if (el.hasAttribute(attr)) {
                const oldVal = el.getAttribute(attr);
                const newVal = replaceDomain(oldVal);
                if (newVal !== oldVal) {
                    el.setAttribute(attr, newVal);
                }
            }
        });

        // Обрабатываем
        if (el.hasAttribute('srcset')) {
            const oldSrcset = el.getAttribute('srcset');
            const newSrcset = oldSrcset
                .split(',')
                .map(part => {
                    const trimmed = part.trim();
                    const i = trimmed.indexOf(' ');
                    if (i === -1) return replaceDomain(trimmed);
                    return replaceDomain(trimmed.slice(0, i)) + trimmed.slice(i);
                })
                .join(', ');

            if (newSrcset !== oldSrcset) {
                el.setAttribute('srcset', newSrcset);
            }
        }

        // Обрабатываем
        if (el.hasAttribute('style')) {
            const oldStyle = el.getAttribute('style');
            const newStyle = oldStyle.replaceAll(FROM, TO);
            if (newStyle !== oldStyle) {
                el.setAttribute('style', newStyle);
            }
        }
    }

    function processAll(root = document) {
        if (!root.querySelectorAll) return;
        root.querySelectorAll('[href], [src], [poster], [srcset], [style]').forEach(processElement);
    }

    // Перехват 
    const originalOpen = window.open;
    window.open = function (url, ...args) {
        return originalOpen.call(this, replaceDomain(url), ...args);
    };

    // Перехват кликов по ссылкам
    document.addEventListener('click', (e) => {
        const link = e.target.closest('a[href]');
        if (!link) return;

        const oldHref = link.getAttribute('href');
        const newHref = replaceDomain(oldHref);
        if (newHref !== oldHref) {
            link.setAttribute('href', newHref);
        }
    }, true);

    // После загрузки страницы — обработать всё
    document.addEventListener('DOMContentLoaded', () => {
        processAll();
    });

    // Отслеживание новых элементов
    const observer = new MutationObserver((mutations) => {
        for (const mutation of mutations) {
            if (mutation.type === 'attributes') {
                processElement(mutation.target);
            }

            for (const node of mutation.addedNodes) {
                if (node.nodeType !== 1) continue;
                processElement(node);
                processAll(node);
            }
        }
    });

    observer.observe(document.documentElement, {
        childList: true,
        subtree: true,
        attributes: true,
        attributeFilter: ['href', 'src', 'poster', 'srcset', 'style']
    });

    // Дополнительно перехватываем fetch
    const originalFetch = window.fetch;
    window.fetch = function (resource, init) {
        if (typeof resource === 'string') {
            resource = replaceDomain(resource);
        } else if (resource instanceof Request) {
            resource = new Request(replaceDomain(resource.url), resource);
        }
        return originalFetch.call(this, resource, init);
    };

    // Дополнительно перехватываем XMLHttpRequest
    const originalXHROpen = XMLHttpRequest.prototype.open;
    XMLHttpRequest.prototype.open = function (method, url, ...args) {
        return originalXHROpen.call(this, method, replaceDomain(url), ...args);
    };
})();