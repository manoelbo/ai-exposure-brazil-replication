# Occupational Exposure to Artificial Intelligence and Labor-Market Outcomes in Brazil: Replication Package

Este repositório contém a versão 2.0.0 do pacote de replicação da dissertação
de mestrado de Manoel Brasil Orlandi na Fundação Getulio Vargas, Escola de
Economia de São Paulo (FGV EESP). O pacote reproduz todas as tabelas e figuras
computacionais das Seções 3--5 e dos Apêndices A--D. A Tabela 2.1 é uma síntese
da literatura e, intencionalmente, não faz parte do registro computacional.

- Autor: [Manoel Brasil Orlandi](https://orcid.org/0009-0008-2432-812X)
- Afiliação: Fundação Getulio Vargas, Escola de Economia de São Paulo (FGV EESP)

O registro da dissertação contém exatamente 53 publicações. Cada linha de
`config/manuscript_artifacts.csv` identifica seu produtor, insumo analítico,
artefato de referência e comando público. Os números apresentados no texto
são registrados separadamente em `config/numeric_claims.csv` e reconciliados
após cada execução.

## Situação atual: pacote de referência

**Nesta versão, o pacote não distribui os dados.** O bundle analítico e o
cache das fontes brutas não são publicados com o repositório. Por isso, o
pacote funciona como referência: permite consultar como cada base foi
construída, como cada modelo foi estimado e como cada tabela e figura foi
renderizada, e traz em `results/reference/` todos os resultados de referência
que sustentam a dissertação. Com o que está publicado, não é possível
reexecutar a análise de ponta a ponta.

O que está disponível:

- o código completo de construção das bases, estimação, diagnóstico e
  renderização (`code/`, `R/`, `run_replication.py`);
- os contratos de dados, esquemas e hashes esperados (`config/`), que
  documentam exatamente quais insumos cada etapa usa;
- os resultados de referência de todas as tabelas, figuras e diagnósticos
  (`results/reference/`);
- o [guia de leitura do código em R](https://manoelbo.github.io/ai-exposure-brazil-replication/).

As instruções abaixo descrevem como o pacote é executado quando os insumos
estão disponíveis. Elas ficam registradas para documentar o procedimento usado
na dissertação.

## Início rápido

Os requisitos são Python 3.10, `uv`, R 4.4.1 e o bundle analítico descrito em
`DATA_AVAILABILITY.md`, que não é distribuído nesta versão.

```bash
uv sync --frozen
Rscript -e 'if (!requireNamespace("renv", quietly = TRUE)) install.packages("renv"); renv::restore(prompt = FALSE)'

uv run python run_replication.py \
  --target all \
  --mode reproduce \
  --data-dir /path/to/replication-v2-bundle \
  --output-dir results/reproduced
```

O ambiente R bloqueado inclui `fixest` 0.14.0, `HonestDiD` 0.2.6 e
`data.table` 1.17.0. Os contratos de modelo entre linguagens são estimados de
forma independente: o R nunca lê coeficientes produzidos pelo Python. Toda
execução pública falha na verificação preliminar se o R 4.4.1 e as versões dos
pacotes não corresponderem ao `renv.lock`. As rotinas do HonestDiD recebem o
vetor do estudo de evento e a matriz de covariância estimados nessa replicação
independente em R; antes de executar qualquer análise de sensibilidade, o
Python valida esses insumos contra suas próprias estimativas.

O modo `reproduce` funciona offline: lê apenas o bundle analítico congelado,
reestima os resultados registrados, renderiza as publicações, valida a
referência imutável, executa as replicações independentes em R e reconcilia os
números da dissertação. O modo `full` obtém as fontes oficiais ausentes,
verifica seus hashes, reconstrói os insumos derivados e, então, executa o mesmo
DAG inferencial. Uma execução `full` que inclua a PNADc exige a opção explícita
`--billing-project` e falha na verificação preliminar quando ela é omitida.

Para inspecionar o plano completo sem executá-lo:

```bash
uv run python run_replication.py \
  --target all \
  --mode reproduce \
  --data-dir /path/to/replication-v2-bundle \
  --dry-run
```

Para reconstruir todos os componentes a partir do cache congelado das fontes
oficiais:

```bash
uv run python run_replication.py \
  --target all \
  --mode full \
  --data-dir /path/to/replication-v2-bundle \
  --raw-dir /path/to/replication-v2-raw-cache \
  --billing-project YOUR_BILLING_PROJECT
```

## Interface pública

```text
python run_replication.py
  --target {all,section3,caged,rais,pnadc,spatial}
  --mode {reproduce,full}
  [--data-dir PATH]
  [--raw-dir PATH]
  [--output-dir PATH]
  [--billing-project PROJECT]
  [--skip-figures]
  [--dry-run]
```

O alias legado `--section 3|4-5|all` continua disponível. A opção
`--section 4-5` seleciona CAGED, RAIS, PNADc e a análise de suporte espacial.

Os resultados são gravados somente em `results/reproduced/` ou no diretório de
saída solicitado. Um diretório existente só é substituído quando seu manifesto
comprova que ele é uma saída completa criada por este pacote. O código, os
insumos e `results/reference/` nunca são destinos de saída válidos. O diretório
de saída também deve ser disjunto de `--data-dir` e `--raw-dir`: ele não pode
ser igual, conter ou estar contido em nenhuma dessas raízes de entrada.

## Componentes

| Alvo | Cobertura na dissertação | Papel |
|---|---|---|
| `section3` | Seção 3 | Exposição na PNADc 2025T3 e perfil distributivo |
| `caged` | Seções 4, 5.1--5.2 e Apêndices A e C | Painéis, DiD/DDD, estudos de evento, diagnósticos e casos ocupacionais |
| `rais` | Apêndices B.1 e D.1 | Estoque formal, rotatividade e tempo no emprego |
| `pnadc` | Apêndices B.2 e D.2 | Informalidade, trabalho por conta própria, estoques de emprego e rendimento |
| `spatial` | Seção 4.5 e Apêndices B.3--B.4 | Placebos, pré-tendências e regra de parada por suporte |

O componente espacial termina no gate de suporte registrado. A família F
mantém 12 posições declaradas, mas não contém coeficiente de tratamento,
valor-p nominal nem valor-p ajustado por multiplicidade.

## Evidências e interpretação

O código preserva testes de pré-tendências rejeitados, diagnósticos de matrizes
de covariância não semidefinidas positivas, blocos de covariância com posto
deficiente, suporte reduzido e ausência de identificação. Esses são resultados,
não erros que devam ser ocultados. As estimativas nacionais do CAGED e os
contrastes DDD por subgrupo devem ser interpretados como diferenças
exploratórias pós-ChatGPT entre grupos tratados e de controle, pois os
diagnósticos de identificação registrados falham. RAIS e PNADc fornecem
evidências complementares de mensuração, não extensões causais.

O Python realiza a aquisição, a construção dos dados, a orquestração do DAG e
a renderização. O R recebe insumos analíticos sem coeficientes e contratos de
modelo. Ele reestima independentemente cada resultado numérico registrado; em
seguida, o Python combina identificadores estáveis de análise, modelo, termo e
tempo do evento. Coeficientes e erros-padrão devem concordar dentro de `1e-6`;
contagens de observações, clusters, identificadores de amostra, status e
períodos de referência devem coincidir exatamente.

Consulte `RESEARCH_DESIGN.md` para conhecer os estimandos e gates científicos,
e `DATA_AVAILABILITY.md` para obter detalhes sobre as fontes e o bundle.

## Guia de código em R

O [guia de código em R para leitura no navegador](https://manoelbo.github.io/ai-exposure-brazil-replication/)
acompanha a estrutura da dissertação e mostra como são implementados os painéis
analíticos, estimadores, tabelas, figuras e verificações científicas. Este
repositório inclui uma [cópia HTML independente](docs/index.html) e seu arquivo
fonte, `r-companion/dissertation_code_companion.Rmd`. O guia é um recurso de
leitura; a interface canônica de replicação continua sendo
`run_replication.py`.

## Estrutura do repositório

```text
R/                         estimadores independentes em R
code/
  common/                  caminhos, manifestos e validações compartilhados
  section3/                análise descritiva
  caged/                   ingestão, painéis, modelos e auditorias do CAGED
  rais/, pnadc/, spatial/  evidências complementares
  render/                  renderizadores da dissertação
  replication/             registros e verificações entre linguagens
config/                    contratos de publicações, números, análises e dados
data/                      ponto de montagem local; arquivos grandes não são distribuídos
results/reference/         referência imutável assinada
results/reproduced/        saída pública padrão
tests/                     testes científicos, de interface e de portabilidade
```

## Verificação

```bash
OPENBLAS_NUM_THREADS=1 \
OMP_NUM_THREADS=1 \
MKL_NUM_THREADS=1 \
NUMBA_NUM_THREADS=1 \
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 \
uv run pytest -q tests
```

A suíte completa de testes espera que o bundle analítico esteja montado ou
copiado em `data/`. Os testes da interface do runner e de portabilidade podem
ser executados sem essa montagem; execuções públicas também podem manter o
bundle fora do repositório e informar seu caminho por `--data-dir`.

A suíte de testes verifica o DAG tipado, o conjunto de 53 publicações, os
números da dissertação, os gates científicos, o isolamento das saídas, os
manifestos determinísticos e os contratos entre linguagens. Uma versão não é
válida apenas porque um coeficiente é estatisticamente significativo.

## Citação e licença

Use `CITATION.cff` para citar o pacote. O código do pacote e a documentação
original são licenciados sob a licença MIT; os conjuntos de dados de origem
permanecem sujeitos aos termos de seus respectivos provedores.

Ao citar o pacote na dissertação, ele é apresentado como referência do código
e dos resultados, e não como um pacote de reexecução com dados incluídos.
