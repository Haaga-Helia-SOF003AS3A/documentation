# Backend-materiaalit ja julkaisuohjeet Rahtiin

Tämä repositorio sisältää sekä uudet markdown-versiot että alkuperäiset pdf-, docx- ja pptx-tiedostot. Markdown-versioiden yhteydessä on alkuperäisistä dokumenteista tallennetut kuvat `imgs` hakemistossa.

Markdown-versiot on käännetty käyttäen `markitdown` ohjelmaa ([linkki](https://github.com/microsoft/markitdown)). Ohjelma toimii paremmin, kun input on .docx tai .pptx verrattuna .pdf-tiedostoon. Peruskäyttö:

```shell
markitdown path/to/document.docx -o output/path/document.md
```

Kuvat täytyy erikseen lisätä .md-tiedostoon. Mutta `markitdown` arvaa alt-tekstit ja merkkaa kuvien paikat.

Uusiin .md-versioihin on tehty kosmeettisia muutoksia esitystavan muutoksesta johtuen. Muutamia kirjoitusvirheitä on korjattu.
