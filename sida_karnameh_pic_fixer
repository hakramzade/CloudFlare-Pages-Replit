// ==UserScript==
// @name         Fix Sida Picture
// @namespace    http://tampermonkey.net/
// @version      1.5
// @description  Match studentId and fix student images from XHR/Fetch JSON paths on sida.medu.ir, show button only when relevant images exist
// @author       Shaxius سیدحسین اکرم زاده اردکانی
// @match        https://sida.medu.ir/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    let capturedData = [];

    function searchForData(obj) {
        if (obj && typeof obj === 'object') {
            if (Array.isArray(obj)) {
                obj.forEach(el => searchForData(el));
            } else {
                let hasPath = (typeof obj.path === 'string' && obj.path.includes('ImageStudent'));
                let hasId   = (obj.studentId !== undefined && obj.studentId !== null);
                if (hasPath && hasId) {
                    capturedData.push({
                        studentId: String(obj.studentId).trim(),
                        path: obj.path
                    });
                }
                for (const key in obj) {
                    searchForData(obj[key]);
                }
            }
        }
    }

    const _open = XMLHttpRequest.prototype.open;
    XMLHttpRequest.prototype.open = function(...args) {
        this.addEventListener('load', function() {
            try {
                const ct = this.getResponseHeader("Content-Type") || "";
                if (ct.includes("application/json")) {
                    const json = JSON.parse(this.responseText);
                    searchForData(json);
                }
            } catch (e) {}
        });
        return _open.apply(this, args);
    };

    if (window.fetch) {
        const _fetch = window.fetch.bind(window);
        window.fetch = (...args) => {
            return _fetch(...args).then(resp => {
                try {
                    const ct = resp.headers.get("content-type") || "";
                    if (ct.includes("application/json")) {
                        resp.clone().json().then(json => searchForData(json)).catch(()=>{});
                    }
                } catch (e) {}
                return resp;
            });
        };
    }

    function addButton() {
        let btn = document.getElementById("fixPicBtn");
        if (!btn) {
            btn = document.createElement("button");
            btn.id = "fixPicBtn";
            btn.textContent = "حل مشکل تصویر کارنامه";
            Object.assign(btn.style, {
                position: "fixed",
                top: "10px",
                right: "10px",
                zIndex: "99999",
                padding: "6px 12px",
                background: "#d9534f",
                color: "#fff",
                fontWeight: "bold",
                border: "none",
                borderRadius: "6px",
                cursor: "pointer",
                boxShadow: "0 2px 6px rgba(0,0,0,0.3)",
                display: "none" // hidden by default
            });

            btn.addEventListener("click", () => {
                if (!capturedData.length) {
                    alert("تصویری دریافت نشده.");
                    return;
                }

                let updated = 0;
                const imgs = document.querySelectorAll('img[style="height: 77px; float: left;"]');

                imgs.forEach(img => {
                    let wrapperDiv = img.closest("div");
                    if (wrapperDiv && wrapperDiv.parentElement) {
                        let parentDiv = wrapperDiv.parentElement;
                        if (parentDiv && parentDiv.parentElement) {
                            let mainDiv = parentDiv.parentElement;
                            let tds = mainDiv.querySelectorAll("td");
                            let matched = false;
                            tds.forEach(td => {
                                if (matched) return;
                                let studentIdText = td.textContent.trim();
                                let match = capturedData.find(d => d.studentId === studentIdText);
                                if (match) {
                                    img.src = match.path;
                                    updated++;
                                    matched = true;
                                }
                            });
                        }
                    }
                });

                capturedData.length = 0;
                alert(`${updated} مورد بروز شد`);
            });

            document.body.appendChild(btn);
        }

        const imgsExist = document.querySelector('img[style="height: 77px; float: left;"]') !== null;
        btn.style.display = imgsExist ? "block" : "none";
    }

    document.addEventListener("DOMContentLoaded", addButton);
    setInterval(addButton, 1000);

})();
