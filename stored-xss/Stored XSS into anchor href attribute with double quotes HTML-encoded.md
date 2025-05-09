

## 1. معلومات عامة

| العنصر         | التفاصيل                                                                                                                                                                                    |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **الرابط**     | [Stored XSS into anchor `href` attribute with double quotes HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded) |
| **تاريخ الحل** | 28/4                                                                                                                                                                                        |
| **نوع الثغرة** | ٍStored XSS                                                                                                                                                                                 |

---
## 1. الفكرة العامة لللاب

- الفكرة اان الموقع فيه مكان انا بحط فيه كومنت 

![[Pasted image 20250428005732.png]]

- ف جربت قيم زي اللي فالصورة و لما بصيت ف ال source code 

![[Pasted image 20250428005919.png]]
- زي منتا شايف كل العلامات اتعملها decoding  بس هل كدا خلاص ؟
- لا لسه معاان الرايط ممكن نحط فيه ال payload 
---
# فكرة الهجوم 

- احنا ممكن نشغل كود JavaScript من خلال اللينك عن طريق `javascript:alert(0)`  
```html
<a id="author" href="[javascript:alert(0)](about:blank)">z3rob</a>
```

زي منتا شايف اي حد هيضغط على اللينك هيشغل الكود 
![[Recording 2025-04-28 010739.mp4]]