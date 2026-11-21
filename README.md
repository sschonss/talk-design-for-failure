# Design for Failure — slides

**Slides (PT):** https://sschonss.github.io/talk-design-for-failure/  
**Slides (EN):** https://sschonss.github.io/talk-design-for-failure/en/  
**PDF (PT):** https://sschonss.github.io/talk-design-for-failure/design-for-failure.pdf  
**PDF (EN):** https://sschonss.github.io/talk-design-for-failure/en/design-for-failure.pdf

Deck em [Marp](https://marp.app) baseado no artigo
[Design for Failure: como assim meu sistema falha?](https://luizschons.com/pt-br/posts/design-for-failure/)
e na palestra apresentada na SoftWeek, no PHPeste e no TLC Share Day (2025).

- `slides.md` — o deck em português (32 slides, em 4 partes). Pouco texto por slide; o roteiro fica nas **notas do apresentador** (`<!-- ... -->`).
- `slides.en.md` — the English translation (same 32 slides).
- `themes/aifa.css` — o mesmo tema da talk AI-Friendly Architecture.
- `assets/` — foto e QR codes do artigo.

O deploy no GitHub Pages roda a cada push na `main` (`.github/workflows/pages.yml`).

## Comandos

```bash
npm run dev     # servidor com preview ao vivo
npm run build   # dist/index.html (modo apresentador: tecla P)
npm run pdf     # dist/design-for-failure.pdf
npm run pptx    # dist/design-for-failure.pptx
```
