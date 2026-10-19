# Constituição do Projeto spec01-00

## Princípios Fundamentais

### I. Front-end Puro (Sem Backend)

- A aplicação DEVE ser composta exclusivamente de artefatos estáticos (HTML, CSS, JavaScript,
  assets) executados inteiramente no navegador.
- É PROIBIDO criar, depender de ou exigir qualquer servidor de aplicação próprio, API própria,
  banco de dados remoto, função serverless ou processo de backend de qualquer natureza.
- A aplicação DEVE funcionar quando servida por qualquer hospedagem estática (ou aberta
  localmente via servidor estático de desenvolvimento) sem configuração adicional de servidor.
- Toda regra de negócio, validação e cálculo DEVE ocorrer no cliente.

**Justificativa**: elimina custo e complexidade operacional, garante portabilidade total e
mantém o escopo do projeto concentrado na experiência do usuário.

### II. Persistência Estritamente Client-Side

- Todo dado persistido DEVE residir no dispositivo do usuário, usando exclusivamente APIs do
  navegador: `localStorage`, `sessionStorage`, IndexedDB, Cache Storage ou equivalentes.
- É PROIBIDO enviar dados do usuário a serviços remotos para fins de armazenamento ou
  sincronização.
- O acesso à persistência DEVE ser encapsulado em uma camada de armazenamento única (módulo
  dedicado); componentes de UI NÃO DEVEM chamar APIs de storage diretamente.
- Todo dado persistido DEVE ter esquema versionado; mudanças de formato DEVEM incluir rotina de
  migração ou descarte seguro dos dados antigos.
- A aplicação DEVE tratar a indisponibilidade do storage (modo privado, cota excedida, dados
  corrompidos) sem quebrar, degradando de forma controlada e informando o usuário quando
  pertinente.

**Justificativa**: coerente com o Princípio I, preserva a privacidade do usuário e torna o
comportamento de persistência previsível e testável.

### III. Distribuição Probabilística Testável

- Toda lógica estocástica (sorteios, embaralhamentos, pesos, probabilidades, eventos aleatórios)
  DEVE obter aleatoriedade de uma fonte injetável (gerador de números aleatórios abstraído por
  interface ou parâmetro).
- É PROIBIDO chamar `Math.random()`, `crypto.getRandomValues()` ou equivalentes diretamente
  dentro da lógica de negócio; essas chamadas DEVEM existir apenas na implementação padrão do
  gerador.
- O projeto DEVE disponibilizar um gerador determinístico com semente (seeded PRNG) para uso em
  testes, reprodução de bugs e depuração.
- Toda função probabilística DEVE ter testes que cubram: (a) resultados exatos sob semente ou
  gerador mockado; (b) casos-limite (probabilidade 0, 1, pesos vazios ou únicos); (c) quando
  aplicável, verificação estatística da distribuição em amostras grandes com tolerância
  explícita.
- Pesos e probabilidades DEVEM ser declarados como dados (configuração), não espalhados como
  literais no código.

**Justificativa**: aleatoriedade não controlada torna falhas irreproduzíveis; injeção de
dependência garante que a lógica seja verificável de forma determinística.

### IV. Responsividade Obrigatória

- Todo componente visual DEVE funcionar e ser utilizável em, no mínimo, três faixas de largura:
  celular (a partir de 320px), tablet (~768px) e desktop (≥1024px).
- O layout DEVE ser construído com abordagem mobile-first e unidades fluidas (`%`, `rem`, `vw`,
  Flexbox, Grid); larguras fixas em pixels só são permitidas com justificativa explícita.
- É PROIBIDA a rolagem horizontal da página em qualquer faixa suportada.
- Alvos de toque DEVEM ter no mínimo 44×44px; textos DEVEM permanecer legíveis sem zoom.
- Imagens e mídias DEVEM se adaptar ao contêiner (`max-width: 100%`, `srcset`/`picture` quando
  aplicável).
- Nenhum componente é considerado concluído sem verificação nas três faixas de largura.

**Justificativa**: o público acessa por dispositivos variados; responsividade tratada como
requisito, não como ajuste posterior, evita retrabalho.

### V. Assets Rigorosamente Organizados em Disco

- Todo asset estático DEVE residir em pasta dedicada ao seu tipo, sob um diretório raiz de
  assets único:
  - `assets/images/` — imagens e ícones (raster e SVG);
  - `assets/fonts/` — arquivos de fonte;
  - `assets/data/` — dados estáticos (JSON, CSV, tabelas de probabilidade, conteúdo);
  - `assets/audio/` e `assets/video/` — mídias, quando existirem.
- É PROIBIDO deixar assets soltos na raiz do projeto, misturados ao código-fonte ou em pastas
  de tipo incorreto.
- Nomes de arquivo DEVEM usar `kebab-case`, minúsculas, sem espaços nem acentos.
- Subpastas por domínio ou funcionalidade são permitidas dentro de cada pasta de tipo
  (ex.: `assets/images/cards/`).
- Assets não referenciados DEVEM ser removidos; assets de terceiros DEVEM ter sua origem e
  licença registradas.

**Justificativa**: estrutura previsível facilita manutenção, revisão, cache e evita
referências quebradas.

## Regras de Arquitetura

- **Camadas**: o código DEVE separar, no mínimo, (1) UI/componentes, (2) lógica de domínio pura,
  (3) serviços de infraestrutura do cliente (storage, gerador aleatório). A lógica de domínio
  NÃO DEVE depender de DOM, storage ou fontes de aleatoriedade concretas.
- **Dependências externas**: bibliotecas de terceiros são permitidas somente se executarem no
  navegador e não exigirem backend. Recursos remotos (CDN, fontes externas) DEVEM ter
  alternativa local em `assets/` ou justificativa documentada.
- **Sem segredos**: por não haver backend, é PROIBIDO incluir chaves de API privadas, tokens ou
  credenciais no código ou nos assets.
- **Dados estáticos**: conteúdo e configurações carregados em tempo de execução DEVEM vir de
  `assets/data/` e ser validados no carregamento.
- **Build**: se houver etapa de build, sua saída DEVE ser exclusivamente estática e preservar a
  organização de assets do Princípio V.

## Padrões de Código

- **Testabilidade**: funções de domínio DEVEM ser puras sempre que possível, recebendo
  dependências (storage, gerador aleatório, relógio) por parâmetro ou injeção.
- **Testes obrigatórios**: toda lógica de domínio, toda função probabilística (Princípio III) e
  toda rotina de persistência/migração (Princípio II) DEVEM ter testes automatizados.
- **Testes não determinísticos são proibidos**: nenhum teste pode depender de aleatoriedade não
  semeada, de horário real ou de estado residual de storage; o storage DEVE ser isolado ou
  mockado em cada teste.
- **Estilos**: CSS DEVE seguir mobile-first, com breakpoints centralizados em variáveis ou
  tokens únicos do projeto.
- **Referência a assets**: caminhos de assets DEVEM ser relativos ao diretório `assets/` e,
  preferencialmente, centralizados em um módulo de constantes.
- **Nomenclatura**: nomes de arquivos de código e assets em `kebab-case`; identificadores
  seguem a convenção idiomática da linguagem adotada, aplicada de forma consistente.
- **Simplicidade**: complexidade adicional (novas camadas, bibliotecas, abstrações) DEVE ser
  justificada no plano da funcionalidade.

## Governança

- **Supremacia**: esta constituição é a norma suprema do projeto e prevalece sobre qualquer
  outra prática, preferência, especificação, plano ou tarefa. Em caso de conflito, a
  constituição vence e o artefato conflitante DEVE ser corrigido.
- **Imutabilidade dos Princípios Fundamentais**: os Princípios I a V são imutáveis. NÃO podem
  ser removidos, enfraquecidos, flexibilizados ou suspensos, nem mesmo de forma temporária ou
  pontual. Exceções não são concedidas.
- **Emendas permitidas**: somente emendas que esclareçam redação (PATCH) ou que adicionem regras
  compatíveis e mais restritivas (MINOR) são aceitas. Toda emenda DEVE ser documentada, com
  justificativa, atualização do Sync Impact Report e da linha de versão.
- **Versionamento**: segue SemVer — MAJOR para redefinição incompatível (vedada aos Princípios
  Fundamentais), MINOR para novas seções ou regras ampliadas, PATCH para esclarecimentos.
- **Conformidade**: toda especificação (`/speckit-specify`), plano (`/speckit-plan`) e lista de
  tarefas (`/speckit-tasks`) DEVE passar pelo Constitution Check contra os cinco princípios.
  Toda revisão de código DEVE verificar a conformidade; violações bloqueiam a entrega.

**Version**: 1.0.0 | **Ratified**: 2026-10-19 | **Last Amended**: 2026-10-19
