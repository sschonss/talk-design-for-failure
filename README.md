# Design for Failure — slides

**Slides (PT):** https://sschonss.github.io/talk-design-for-failure/  
**Slides (EN):** https://sschonss.github.io/talk-design-for-failure/en/  
**PDF (PT):** https://sschonss.github.io/talk-design-for-failure/design-for-failure.pdf  
**PDF (EN):** https://sschonss.github.io/talk-design-for-failure/en/design-for-failure.pdf

Deck em [Marp](https://marp.app): versão no tema aifa da palestra original de 2025
([sschonss/design-for-failure](https://github.com/sschonss/design-for-failure), 104 slides), com o mesmo conteúdo, slide a slide.

- `slides.md` — o deck em português (104 slides, os mesmos do original).
- `slides.en.md` — the English translation (same 104 slides; code unchanged).
- `themes/aifa.css` — o mesmo tema da talk AI-Friendly Architecture.
- `assets/` — foto, diagramas originais, logos e QR codes.

O deploy no GitHub Pages roda a cada push na `main` (`.github/workflows/pages.yml`).

## Comandos

```bash
npm run dev     # servidor com preview ao vivo
npm run build   # dist/index.html (modo apresentador: tecla P)
npm run pdf     # dist/design-for-failure.pdf
npm run pptx    # dist/design-for-failure.pptx
```
