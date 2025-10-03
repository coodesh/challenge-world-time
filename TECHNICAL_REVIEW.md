# Revisão Técnica e Melhorias - WorldsTime

Este documento reúne observações sobre possíveis bugs, inconsistências de lógica, práticas de estado e sugestões de melhoria no projeto WorldsTime.

## 1. Gerenciamento de Preferências (PreferencesProvider)
- Correção do flag `loaded`: o `return` dentro do `try` de `matchMedia` impede que `setLoaded(true)` seja executado em cenários normais, bloqueando a persistência de preferências.
- Consolidar lógica de efeitos: separar carregamento inicial, detecção de `prefers-reduced-motion` e listener de `matchMedia` em `useEffect`s distintos, garantindo cleanup adequado e evitando retornos prematuros.
- Cleanup do `matchMedia`: remover listeners órfãos no cleanup do hook para evitar vazamentos de listener.
- Validação de dados do `localStorage`: validar esquema e tipos ao parsear JSON para evitar sobrescrever preferências com dados inválidos.
- Extrair geração de CSS custom properties para módulo utilitário ou hook específico, reduzindo complexidade no provider.
- Utilizar `useCallback` e `useMemo` para funções e valores estáticos (e.g., `accentMap`), evitando recriação desnecessária em cada render.
- Atualização funcional de estado em callbacks para evitar closures obsoletas.

## 2. Sincronização de Tempo (useAccurateUtcTime)
- Falha silenciosa na sincronização de rede: registrar logs ou expor estado de erro quando a chamada `fetch` ao WorldTime API falhar.
- Consolidar timers: unificar `setInterval` de tick e de resync em um único hook ou extrair a lógica de drift em hook dedicado.
- AbortController no cleanup: garantir o cancelamento de requests pendentes em desmontagem do hook.
- Remover estado `tick` desnecessário e usar `requestAnimationFrame` ou `setInterval` único para atualizar o relógio.
- Expor status (`loading`, `error`) no hook para permitir feedback na UI.

## 3. Alarme (AlarmOverlay)
- Arquivo de áudio ausente: referência a `/alarm.mp3` inexistente em `public`, causando falhas de carregamento.
- Cleanup do objeto `Audio`: usar `useRef` e pausar/liberar no cleanup do `useEffect`, evitando instâncias pendentes após desmontagem.
- Lógica de play/pause idempotente: checar `audio.paused` antes de chamar `play()`, reduzindo rejeições silenciosas.
- Extrair lógica de alarme para hook (`useAlarm`), separando estado, timers e efeitos de apresentação.
- Adicionar fallback visual e aria-alert para casos de falha de reprodução.

## 4. SSR e Hidratação (WorldTimeDisplay)
- Uso de `suppressHydrationWarning`: documentar justificativa ou evitar flash de conteúdo usando `next/dynamic` com `ssr: false` ou hook de hidratação.
- Memoização de listas estáticas (`timeZones`, `filtered`) e constantes (`PAGE_SIZE`, `baseTimeZones`) fora do componente para evitar recomputação.
- Revisar `useEffect` de `countryFilter` com `eslint-disable`; incluir todas as dependências ou mover lógica para hook customizado.
- Separar detecção de timezone inicial em hook (`useInitialTimezone`) fora do componente.
 - Extrair formatação de tempo em função utilitária e memoizar opções de `Intl.DateTimeFormat`.
 - Corrigir lógica de paginação: ajustar reset e limites de `page` para evitar páginas vazias e garantir navegação consistente ao filtrar ou alterar o conjunto.

## 5. Desempenho e Otimização
- Lazy-load de componentes pesados (`SettingsPanel`, `AlarmOverlay`, `AmbientBackground`) via `next/dynamic` para reduzir bundle inicial.
- Debounce na busca (`query`) em `WorldTimeDisplay` para reduzir re-renderizações por tecla.
- Aplicar `React.memo`, `useMemo` e `useCallback` em componentes estáticos ou callbacks para evitar renders desnecessários.
- Extrair estilos inline complexos (animações, transform) para classes CSS ou hooks, reduzindo alocação de objetos.

## 6. Qualidade de Código e Boas Práticas
- Habilitar regras estritas de ESLint/TypeScript (`noImplicitAny`, `strictNullChecks`, `react-hooks/exhaustive-deps`) para capturar erros de estado e dependências faltantes.
- Padronizar nomeclatura de variáveis de estado e handlers; evitar uso genérico de `any` em funções de atualização.
- Centralizar configurações e constants (timezones, cores, tamanhos) em módulos dedicados.
- Remover código comentado e importações não utilizadas para manter o código limpo.

## 7. Testes e Cobertura
- Adicionar testes unitários para hooks (`useAccurateUtcTime`, `usePreferences`, `useAlarm`) usando Jest e [react-hooks-testing-library].
- Criar testes de integração para fluxo de persistência de preferências e alarme, simulando timers e `localStorage`.
- Mock de APIs (`Intl`, `matchMedia`, `fetch`) em testes para garantir comportamentos consistentes.

## 8. Acessibilidade (a11y)
- Garantir `role` e `aria-attributes` adequados em modais e overlays (`SettingsPanel`, `AlarmOverlay`, search overlay).
- Adicionar texto alternativo (`alt`) em ícones e imagens estáticas.
- Validar contraste de cores em modo `highContrast` e temas customizados.
- Suporte completo a teclado (tabindex, keyDown) em componentes interativos.

## 9. Segurança e Dependências
- Atualizar dependências críticas (`date-fns-tz`, `framer-motion`, `next`) para versões LTS e corrigir vulnerabilidades com `npm audit`.
- Checar disponibilidade de `window` antes de usar `localStorage` em ambientes sem DOM.

---

**Conclusão:** aplicar essas melhorias reforça a robustez, manutenibilidade e experiência do usuário na aplicação WorldsTime.
