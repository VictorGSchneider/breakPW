# breakPW — Brute force de código numérico (8 dígitos) em **C/OpenMP** e **Go**

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)

> **Uso ético/educacional**: este projeto existe para estudar busca exaustiva (brute force), paralelismo e avaliação de desempenho. O “alvo” é um número informado localmente. **Não** use este código para tentar acessar serviços/sistemas reais.

---

## ✨ O que tem aqui

* Implementação **sequencial** e **paralela** (OpenMP) em **C**.
* Implementação **sequencial** e **paralela** (goroutines) em **Go**.
* **Makefile** para compilar tudo de uma vez.
* Scripts `run_*.sh` que executam **20 vezes** cada binário e geram **CSV** com tempos (em segundos) usando o tempo **interno** impresso pelo programa (`ELAPSED_SEC`).
* Planilha `benchmark_times.xlsx` (opcional) para consolidar os resultados (médias, desvio, speedup etc.).

O espaço de busca é `0..99_999_999` (10^8 possibilidades). O programa pára no momento em que encontra o **alvo** informado na linha de comando.

---

## 🧱 Requisitos

* Linux/macOS (testado no Linux). Em Windows, recomenda‑se **WSL2**.
* **GCC** (com suporte a OpenMP) e `make`
  Ubuntu/Debian: `sudo apt install build-essential`
  (Se usar Clang, troque `CC=gcc` no Makefile **ou** instale `libomp-dev`.)
* **Go** 1.17+ (testado com Go 1.20+)
* `awk` (para os scripts `run_*.sh` extraírem `ELAPSED_SEC`)

---

## 📁 Estrutura

```
breakPW/
├─ Makefile
├─ c/
│  ├─ seq_bruteforce.c         # C sequencial
│  └─ par_bruteforce_omp.c     # C paralelo (OpenMP)
├─ cmd/
│  ├─ go_seq/
│  │  └─ main.go               # Go sequencial
│  └─ go_par/
│     └─ main.go               # Go paralelo (goroutines)
├─ bin/                        # executáveis gerados pelo Make
├─ run_seq.sh                  # 20 execs (C seq) → CSV
├─ run_par.sh                  # 20 execs (C omp) → CSV
├─ run_go_seq.sh               # 20 execs (Go seq) → CSV
├─ run_go_par.sh               # 20 execs (Go par) → CSV
└─ benchmark_times.xlsx        # (opcional) planilha para consolidação
```

---

## 🛠️ Build

```bash
make            # compila bin/seq, bin/par, bin/go_seq, bin/go_par
# alvos específicos
make c          # só C
make go         # só Go
make clean      # remove bin/
```

> Se aparecer “**Clock skew detected**”, é apenas diferença de timestamp. Rode `find . -type f -print0 | xargs -0 touch && make clean && make`.

### Go (módulos)

Se o Go reclamar de módulo ausente, inicialize um **módulo no raiz** do projeto:

```bash
go mod init breakpw
go mod tidy
```

---

## ▶️ Execução

Alvo (8 dígitos) como primeiro argumento; no paralelo também informe **threads/workers**.

```bash
# Sequenciais
./bin/seq     99999999
./bin/go_seq  99999999

# Paralelos
./bin/par     99999999 8    # OpenMP com 8 threads
./bin/go_par  99999999 8    # Go com 8 workers (GOMAXPROCS=8)
```

Saída típica:

```
FOUND 99999999
THREADS 8             # só nas versões paralelas
ELAPSED_SEC 0.042157
```

> **Dica**: use **99999999** para simular o “pior caso” (evita tempos \~0s quando o alvo está no começo). Os binários imprimem o tempo **interno** com 6 casas decimais.

---

## ⏱️ Benchmarks automatizados (20 execuções)

Os scripts abaixo coletam 20 execuções e gravam CSV com os tempos internos do programa (`ELAPSED_SEC`). O alvo padrão é **99999999** para gerar tempos significativos.

```bash
chmod +x run_*.sh

# C
./run_seq.sh                 # → times_seq.csv
./run_par.sh 99999999 8      # → times_par_8t.csv

# Go
./run_go_seq.sh              # → times_go_seq.csv
./run_go_par.sh 99999999 8   # → times_go_par_8w.csv
```

Cada CSV tem o formato:

* `times_seq.csv`: `run,seconds`
* `times_par_8t.csv`: `run,threads,seconds`
* `times_go_seq.csv`: `run,seconds`
* `times_go_par_8w.csv`: `run,workers,seconds`

### Planilha (Excel)

Abra `benchmark_times.xlsx` e preencha as abas conforme os CSVs. A aba **Summary** calcula média, desvio, min, max, mediana e **speedup** automaticamente.

> Se o Excel (pt‑BR) tratar os números como **texto**: selecione a coluna, **Dados → Texto para colunas → Avançado → decimal ponto** ou use fórmulas `VALOR(SUBSTITUIR(...))` na **Summary**.

---

## ⚙️ Parâmetros e detalhes técnicos

* **Espaço de busca**: `0..99_999_999` (10^8). Comparação direta `i == target`.
* **OpenMP (C)**: laço `#pragma omp parallel for schedule(static)`, sinalização de parada com *flag* compartilhada.
* **Go paralelo**: divisão por *stride* (cada worker começa em um deslocamento e incrementa de `workers` em `workers`), finalização concorrente com `sync/atomic`.
* **Tempo interno**: `clock_gettime(CLOCK_MONOTONIC)` no C; `time.Now()` no Go.

---

## 📈 Boas práticas de medição

* Feche outros programas e **mantenha o notebook no carregador**.
* Linux: modo desempenho (se disponível): `sudo cpupower frequency-set -g performance`.
* Meça com **vários valores** de threads/workers (1,2,4,8,16…) e reporte **speedup**.

---

## 🩺 Solução de problemas

* **Tempos \~0s**: use alvo `99999999` e/ou scripts `run_*.sh` (usam o tempo interno com 6 casas decimais).
* **Go: “cannot find main module”**: `go mod init breakpw && go mod tidy` no raiz do projeto.
* **VS Code: “C source files not allowed … / main redeclared”**: mantenha C em `c/` e cada binário Go em `cmd/<nome>/` (já está assim neste repo).
* **OpenMP não reconhecido**: garanta que está compilando com **GCC** (`CC=gcc`) e com `-fopenmp`; no Clang, instale `libomp-dev` ou use GCC.
* **Clock skew detected**: `find . -type f -print0 | xargs -0 touch && make clean && make`.

---

## 📄 Licença

Distribuído sob a **GNU General Public License v3.0 (GPL‑3.0)**. Consulte o arquivo [`LICENSE`](LICENSE) na raiz do repositório para o texto integral da licença.

---

## 🧪 Exemplo de rodada completa

```bash
make clean && make
./run_seq.sh
./run_par.sh 99999999 8
./run_go_seq.sh
./run_go_par.sh 99999999 8
```

Importe os CSVs na planilha e confira a aba **Summary** para as métricas e o **speedup**.

---

## 👥 Créditos

Implementação e scripts criados para fins didáticos na disciplina de **Programação Paralela**. Sugestões e PRs são bem‑vindos! :)
