# Botão `severity="secondary"` apagado no tema light

- **Data:** 2026-09-03
- **Tags:** primevue, vue, tailwind, tema, button
- **Status:** resolvido

## Contexto

PrimeVue 4.5 (styled mode) com o preset `Aura` stock (`resources/js/app.ts`,
config do `PrimeVue`), Tailwind v4 + `tailwindcss-primeui`. Tema light. Botões
de contexto secundário (`<Button severity="secondary">`) — ex.
`resources/js/components/AppTopbar.vue`, `resources/js/components/UserMenu.vue`.

## Sintoma

No tema claro, o botão secondary fica quase sem cor de fundo: o cinza é tão
próximo do branco que o botão parece um `text`/`plain`, sem hierarquia visual
contra o `bg-surface-0` da página. No tema dark o contraste é adequado.

## Causa raiz

O token stock do Aura `button.colorScheme.light.secondary.background` resolve
para `{surface.100}`, que sobre `surface.0` tem contraste muito baixo. O projeto
não customiza o preset (não há `definePreset`), então herda esse valor.

## Solução

Padrão do projeto, sem alterar o preset: ao criar botão de contexto secundário
que precise de presença visual no tema light, reforçar o fundo com utilitário
Tailwind. A ordem de camadas CSS do projeto
(`theme, base, primevue, components, utilities` — `app.ts` e
`resources/css/app.css`) garante que `bg-*` do Tailwind vence o token do Aura:

```vue
<Button
    label="..."
    severity="secondary"
    class="bg-surface-200 hover:bg-surface-300 dark:bg-transparent dark:hover:bg-surface-800"
/>
```

O `dark:` mantém o comportamento padrão do Aura no tema escuro.

Alternativa global (fora do escopo quando esta entrada foi criada, mas é o
caminho se o incômodo se repetir): `definePreset(Aura, { ... })` em `app.ts`
ajustando `components.button.colorScheme.light.secondary.background` para
`{surface.200}` e `hover.background` para `{surface.300}` — resolve de uma vez
para todos os botões secondary.

## Como evitar ou detectar antes

- Ao revisar tela nova no tema light, conferir se botões secundários têm fundo
  distinguível do fundo da página; se o design pede destaque e o botão some,
  aplicar a classe acima.
- `grep -rn 'severity="secondary"' resources/js` para inventariar os botões
  afetados antes de decidir entre ajuste pontual e `definePreset`.

## Referências

- `resources/js/app.ts` (config do PrimeVue, `cssLayer.order`)
- `resources/css/app.css` (`@layer` order, `@plugin 'tailwindcss-primeui'`)
- `resources/js/components/AppTopbar.vue`
- `resources/js/components/UserMenu.vue`
- https://primevue.org/theming/styled/
