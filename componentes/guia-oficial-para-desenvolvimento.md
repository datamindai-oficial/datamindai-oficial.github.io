# Padrão de Componentes de Software — Guia Oficial

> **Objetivo:** padronizar a criação, documentação, empacotamento, testes, versionamento e demonstração de *componentes de software* (módulos Python com classe e/ou funções) para acelerar o desenvolvimento, facilitar reuso e garantir qualidade.

Um **componente** é um módulo autocontido (ex.: `converter.py`) que expõe **classe(s)** e/ou **funções** com entradas/saídas bem definidas, documentação completa, **CHANGELOG**, testes mínimos e um **Jupyter Notebook de demonstração**. Cada componente inclui **README.md**, **requirements.txt** e metadados de **versão**/**suporte a Python**.

---

## Sumário

* [1) Estrutura de Pastas](#1-estrutura-de-pastas)
* [2) Convenções de Código (PEP 8/257)](#2-convenções-de-código-pep-8257)
* [3) Contrato de Interface](#3-contrato-de-interface)
* [4) Documentação em Código](#4-documentação-em-código)
* [5) README.md — Template](#5-readmemd--template)
* [6) requirements.txt — Diretrizes](#6-requirementstxt--diretrizes)
* [7) Notebook de Demonstração](#7-notebook-de-demonstração)
* [8) Implementação de Exemplo (Esqueleto)](#8-implementação-de-exemplo-esqueleto)
* [9) Testes Mínimos (pytest)](#9-testes-mínimos-pytest)
* [10) Qualidade & Automação](#10-qualidade--automação)
* [11) Versionamento do Componente](#11-versionamento-do-componente)
* [12) Checklist de Entrega](#12-checklist-de-entrega)
* [13) Exemplos de Nomes](#13-exemplos-de-nomes)
* [14) Como Avaliar um Componente](#14-como-avaliar-um-componente)

---

## 1) Estrutura de Pastas

```
componente_nome/
├─ src/
│  └─ componente_nome/
│     ├─ __init__.py
│     ├─ _version.py                # fonte da verdade da versão
│     └─ converter.py               # código do componente
├─ notebooks/
│  └─ demo_converter.ipynb          # notebook de demonstração
├─ tests/
│  └─ test_converter.py             # testes automáticos (mínimo viável)
├─ README.md                        # objetivo, escopo, uso, limitações
├─ CHANGELOG.md                     # histórico de versões
├─ requirements.txt                 # dependências de runtime
├─ pyproject.toml                   # (recomendado) build/lint/type-check
├─ .pre-commit-config.yaml          # (opcional) automações locais
└─ LICENSE                          # (opcional)
```

> **Estrutura mínima (quando isolado):**
>
> ```
> converter/
> ├─ converter.py
> ├─ demo_converter.ipynb
> ├─ README.md
> ├─ CHANGELOG.md
> └─ requirements.txt
> ```

---

## 2) Convenções de Código (PEP 8/257)

**Estilo de nomes**

* **Módulos/arquivos:** `snake_case` curto e descritivo (`converter.py`).
* **Pacotes:** `snake_case` (`componente_nome`).
* **Funções:** `snake_case` (`pdf_to_csv`).
* **Variáveis:** `snake_case` (`input_path`, `page_range`).
* **Constantes:** `UPPER_SNAKE_CASE` (`DEFAULT_ENCODING`).
* **Classes:** `PascalCase` (`DocumentConverter`).
* **Privado:** prefixo `_` para membros internos (`_cache`).

**Boas práticas**

* *Type hints* em 100% das assinaturas públicas.
* Docstrings padrão **Google** ou **NumPy** em funções/classes/métodos.
* Exceções **específicas** + mensagens claras (não “engolir” exceções).
* **Logging** (`logging`) em vez de `print` em produção.
* Priorize **funções puras**; documente efeitos colaterais.
* Valide **I/O** (paths, permissões, encoding, tamanhos).
* Documente limites de **performance** (memória/tempo esperado).

**Qualidade/Automação**

* **Formatter:** `black`.
* **Linter:** `ruff` (ou `flake8`).
* **Type-checker:** `mypy`.
* **pre-commit:** hooks para formatar/lintar/tipar/notebooks.

---

## 3) Contrato de Interface

**Requisitos**

* Funções públicas declaram **entradas/saídas** (tipos, formatos) e **erros**.
* Classe principal (se houver) encapsula estado **mínimo** e fornece métodos atômicos.
* Conversões (`pdf_to_csv`, `pdf_to_html`, `csv_to_csv`) devem:

  * validar argumentos;
  * retornar **resultado estruturado** (ex.: `Path` do arquivo ou `pd.DataFrame`);
  * não modificar *inputs* originais;
  * aceitar `overwrite: bool = False` quando fizer sentido.

**Exemplo de assinaturas**

```python
from pathlib import Path
from typing import Optional, Sequence

class DocumentConverter:
    """Conversões entre formatos de documento."""

    def __init__(self, default_encoding: str = "utf-8") -> None:
        self.default_encoding = default_encoding

    def pdf_to_csv(
        self,
        input_pdf: Path | str,
        output_csv: Optional[Path | str] = None,
        *,
        pages: Optional[Sequence[int]] = None,
        delimiter: str = ",",
        overwrite: bool = False,
    ) -> Path:
        """Converte PDF em CSV."""
        ...
```

**Hierarquia de erros customizados**

```python
class ComponentError(Exception):
    """Erro base do componente."""

class ValidationError(ComponentError):
    """Entrada inválida, conflito de parâmetros, etc."""

class ConversionError(ComponentError):
    """Falha no processo de conversão."""
```

---

## 4) Documentação em Código

**Docstrings (estilo Google)**

```python
def csv_to_csv(path: Path | str, *, delimiter: str = ",") -> Path:
    """Normaliza um CSV aplicando regras padrão.

    Args:
      path: Caminho do CSV de entrada.
      delimiter: Delimitador atual do arquivo.

    Returns:
      Caminho do CSV normalizado.

    Raises:
      FileNotFoundError: Quando `path` não existe.
      ValidationError: Quando o arquivo não é um CSV válido.
    """
    ...
```

**Comentários**

* Explique o **porquê** (racional), não o óbvio **como**.
* Para trechos não triviais, cite complexidade esperada ou referências.

---

## 5) README.md — Template

````markdown
# Nome do Componente: Converter

## Objetivo
Converte documentos entre formatos (PDF ↔ CSV/HTML, CSV → CSV normalizado) para facilitar análise e ingestão.

## Escopo
- ✅ Extração de tabelas de PDFs estruturados
- ✅ Geração de CSV/HTML a partir de PDF
- ✅ Normalização de CSV (delimitador, encoding, headers)
- ❌ OCR de PDFs somente-imagem (fora de escopo)
- ❌ Parsing de PDFs com layout altamente irregular

## Instalação
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
````

## Suporte a Python

* Testado em: 3.9, 3.10, 3.11
* Mínimo suportado: 3.9

## Dependências

Veja `requirements.txt` para versões exatas.

## Uso Rápido

```python
from componente_nome.converter import DocumentConverter
conv = DocumentConverter()
conv.pdf_to_csv("exemplos/tabela.pdf", output_csv="saida/tabela.csv")
```

## Demonstração

Abra `notebooks/demo_converter.ipynb` e execute as células numeradas.

## Limitações e Não-Suporte

* PDFs somente-imagem exigem OCR (não implementado).
* Arquivos > 200MB podem falhar por memória.
* Codificações não-UTF-8 podem requerer `default_encoding`.

## Erros Comuns

* `FileNotFoundError`: caminho inválido.
* `ValidationError`: conflito de parâmetros/arquivo já existe.
* `ConversionError`: falha de extração por layout inconsistente.

## Versão & Suporte

* Versão atual: **0.1.0**
* Estado: **stable** | experimental | beta | deprecated | archived
* Python: **3.9–3.11**
* Próxima remoção: *nenhuma*

Veja também: [CHANGELOG.md](./CHANGELOG.md).


---

## 6) requirements.txt — Diretrizes

- **Fixe versões** com `==` para reprodutibilidade.  
- Separe **runtime** (este arquivo) de **dev** (em `pyproject.toml` ou `requirements-dev.txt`).

**Exemplo**
```python
pandas==2.2.0
pdfplumber==0.11.0
beautifulsoup4==4.12.3

```

---

## 7) Notebook de Demonstração

**Arquivo:** `notebooks/demo_converter.ipynb`

**Seções mínimas**
1. **Introdução**: objetivo, versões (`python`, libs) e caminhos dos dados de exemplo.
2. **Setup**: imports, `logging`, utilitários para `Path`.
3. **Casos de Uso** (uma célula por função pública):
   - `pdf_to_csv()`
   - `pdf_to_html()`
   - `csv_to_csv()` (normalização)
4. **Tratamento de Erros**: mensagens amigáveis para cenários comuns.
5. **Métricas/Desempenho**: tempos médios em arquivos de exemplo.
6. **Conclusões**: limitações e próximos passos.

> **Boas práticas no notebook**: células curtas, *seed* fixa, sem *prints* excessivos (use `logging`).  
> **Carimbo de versão** no topo:
> ```python
> import platform, sys, pandas as pd
> import componente_nome as pkg
> print("component:", pkg.__version__)
> print("python:", platform.python_version())
> print("pandas:", pd.__version__)
> ```

---

## 8) Implementação de Exemplo (Esqueleto)

```python
# src/componente_nome/converter.py
from __future__ import annotations
from dataclasses import dataclass
from pathlib import Path
from typing import Optional, Sequence
import logging

logger = logging.getLogger(__name__)

class ComponentError(Exception):
    """Erro base do componente."""

class ValidationError(ComponentError):
    """Entrada inválida, conflito de parâmetros, etc."""

class ConversionError(ComponentError):
    """Falha no processo de conversão."""

@dataclass(frozen=True)
class ConversionResult:
    output: Path
    details: dict[str, object] | None = None

class DocumentConverter:
    def __init__(self, default_encoding: str = "utf-8") -> None:
        self.default_encoding = default_encoding

    def _resolve_output(self, input_path: Path, suffix: str, output: Optional[Path]) -> Path:
        if output is None:
            output = input_path.with_suffix(suffix)
        return output

    def pdf_to_csv(
        self,
        input_pdf: Path | str,
        output_csv: Optional[Path | str] = None,
        *,
        pages: Optional[Sequence[int]] = None,
        delimiter: str = ",",
        overwrite: bool = False,
    ) -> Path:
        """Converte PDF em CSV (tabelas estruturadas)."""
        pdf = Path(input_pdf)
        if not pdf.exists():
            raise FileNotFoundError(pdf)
        out = Path(output_csv) if output_csv else self._resolve_output(pdf, ".csv", None)
        if out.exists() and not overwrite:
            raise ValidationError(f"Arquivo já existe: {out}. Use overwrite=True.")
        try:
            # TODO: implementar extração com pdfplumber/tabula/etc.
            logger.info("Extraindo tabelas de %s para %s", pdf, out)
            out.write_text("col1,col2\n1,2\n", encoding=self.default_encoding)
            return out
        except Exception as exc:  # noqa: BLE001
            raise ConversionError(f"Falha ao converter {pdf} -> {out}: {exc}") from exc

    def pdf_to_html(
        self,
        input_pdf: Path | str,
        output_html: Optional[Path | str] = None,
        *,
        overwrite: bool = False,
    ) -> Path:
        pdf = Path(input_pdf)
        if not pdf.exists():
            raise FileNotFoundError(pdf)
        out = Path(output_html) if output_html else self._resolve_output(pdf, ".html", None)
        if out.exists() and not overwrite:
            raise ValidationError(f"Arquivo já existe: {out}. Use overwrite=True.")
        try:
            logger.info("Convertendo %s para HTML em %s", pdf, out)
            out.write_text("<html><body><p>Demo</p></body></html>", encoding=self.default_encoding)
            return out
        except Exception as exc:  # noqa: BLE001
            raise ConversionError(f"Falha ao converter {pdf} -> {out}: {exc}") from exc

    def csv_to_csv(
        self,
        path: Path | str,
        *,
        delimiter: str = ",",
        overwrite: bool = False,
    ) -> Path:
        csv = Path(path)
        if not csv.exists():
            raise FileNotFoundError(csv)
        out = csv if overwrite else csv.with_name(csv.stem + ".normalized.csv")
        try:
            logger.info("Normalizando CSV %s -> %s", csv, out)
            # TODO: ler e reescrever com pandas.
            text = csv.read_text(encoding=self.default_encoding)
            out.write_text(text, encoding=self.default_encoding)
            return out
        except Exception as exc:  # noqa: BLE001
            raise ConversionError(f"Falha ao normalizar {csv}: {exc}") from exc
````

---

## 9) Testes Mínimos (pytest)

```python
# tests/test_converter.py
from pathlib import Path
import pytest
from componente_nome.converter import DocumentConverter, ValidationError

def test_pdf_to_csv_raises_when_missing(tmp_path: Path):
    conv = DocumentConverter()
    with pytest.raises(FileNotFoundError):
        conv.pdf_to_csv(tmp_path/"missing.pdf")

def test_pdf_to_csv_overwrite_flag(tmp_path: Path):
    pdf = tmp_path/"in.pdf"
    pdf.write_text("demo", encoding="utf-8")
    conv = DocumentConverter()
    out = conv.pdf_to_csv(pdf)
    assert out.exists()
    with pytest.raises(ValidationError):
        conv.pdf_to_csv(pdf, output_csv=out)
```

---

## 10) Qualidade & Automação

**`pyproject.toml` (trechos)**

```toml
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "componente-nome"
version = "0.1.0"
requires-python = ">=3.9"
dependencies = [
  "pandas==2.2.0",
  "pdfplumber==0.11.0",
  "beautifulsoup4==4.12.3",
]

[tool.black]
line-length = 100
target-version = ["py39","py310","py311"]

[tool.ruff]
line-length = 100
select = ["E","F","I","UP","BLE"]

[tool.mypy]
python_version = "3.11"
strict = true
ignore_missing_imports = true
```

**Pre-commit** (`.pre-commit-config.yaml`)

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.4.2
    hooks: [ { id: black } ]
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.5.0
    hooks: [ { id: ruff } ]
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks: [ { id: mypy } ]
  - repo: https://github.com/kynan/nbstripout
    rev: 0.7.1
    hooks: [ { id: nbstripout } ]
```

---

## 11) Versionamento do Componente

> **Objetivo:** garantir previsibilidade de mudanças, rastreabilidade de releases e clareza sobre suporte/compatibilidade.

### 11.1 Política de Versão (SemVer)

Use **SemVer**: `MAJOR.MINOR.PATCH`.

* **MAJOR (X)**: mudanças **incompatíveis** na API pública.
* **MINOR (Y)**: **novas funcionalidades** compatíveis.
* **PATCH (Z)**: **correções** sem alterar API.

**Compatibilidade**

* API pública = funções/classes documentadas no README/docstrings.
* Alterações em **nomes, assinaturas, contratos, exceções** ou **comportamentos documentados** ⇒ **MAJOR**.
* **Depreciação** obrigatória por **≥1 MINOR** antes de remoção.
* Recursos **experimental** podem mudar sem estabilidade (declarar explicitamente).

### 11.2 Identidade de Versão (código, pacote e CLI)

```python
# src/componente_nome/__init__.py
from ._version import __version__

# src/componente_nome/_version.py
__version__ = "0.1.0"
```

Se houver CLI:

```python
def main() -> None:
    import argparse
    from componente_nome import __version__
    parser = argparse.ArgumentParser("componente-nome")
    parser.add_argument("--version", action="store_true", help="Exibe a versão")
    args = parser.parse_args()
    if args.version:
        print(__version__)
        return
```

No `pyproject.toml`, mantenha coerência:

```toml
[project]
version = "0.1.0"
requires-python = ">=3.9"
```

> Preferencialmente, mantenha **uma fonte da verdade** e derive as demais.

### 11.3 Ciclo de Vida

| Estado       | Critério                                                 | Expectativa       |
| ------------ | -------------------------------------------------------- | ----------------- |
| experimental | API sujeita a alterações bruscas; documentação explícita | alta              |
| beta         | API quase estável; faltam coberturas e métricas          | média             |
| stable       | API estável, cobertura adequada, métricas/SLIs           | baixa             |
| deprecated   | Data de remoção anunciada                                | remoção N+1 MINOR |
| archived     | Sem manutenção                                           | nenhuma           |

Declare o estado no **README** (badge + seção “Status”).

### 11.4 Matriz de Compatibilidade

Exemplo:

```markdown
### Compatibilidade

| Versão do Componente | Python  | pandas | SO testado     |
|----------------------|---------|--------|----------------|
| 0.3.x                | 3.10–3.12 | 2.2.x | Ubuntu 22.04   |
| 0.2.x                | 3.9–3.11  | 2.1.x | Ubuntu 20.04   |
```

Teste em **matriz de CI** (3.9–3.12).

### 11.5 Changelog (Keep a Changelog)

Arquivo `CHANGELOG.md`:

```markdown
# Changelog
Formato baseado em Keep a Changelog, aderente a SemVer.

## [0.1.0] - 2025-10-06
### Added
- Versão inicial com `pdf_to_csv`, `pdf_to_html`, `csv_to_csv`
```

### 11.6 Depreciação e Remoção

* Anuncie no **CHANGELOG** e no **README** (tabela “Remoções programadas”).
* Emita `DeprecationWarning`:

```python
import warnings
def pdf_to_csv(..., delimiter: str | None = ",", **kwargs):
    if delimiter is None:
        warnings.warn(
            "delimiter=None está depreciado e será removido em 0.4.0; use ','",
            category=DeprecationWarning,
            stacklevel=2,
        )
```

* Janela mínima: **≥1 MINOR** até remoção.

### 11.7 Estratégia de Releases

**Tags e Releases**

```bash
git tag -a v0.1.0 -m "Release 0.1.0"
git push origin v0.1.0
```

**Commits convencionais (recomendado)**

* `feat:` → MINOR
* `fix:` → PATCH
* `feat!:`/`refactor!:` → MAJOR
* `docs:`, `chore:`, `build:`, `test:` não alteram versão por si.

**Bump de versão (escolha 1)**

* Hatch: `hatch version minor`
* Poetry: `poetry version minor`
* bump2version: ver `.bumpversion.cfg`:

```ini
[bumpversion]
current_version = 0.1.0
commit = True
tag = True
tag_name = v{new_version}

[bumpversion:file:pyproject.toml]
search = version = "{current_version}"
replace = version = "{new_version}"

[bumpversion:file:src/componente_nome/_version.py]
```

### 11.8 Pipeline de Release (GitLab CI) — Exemplo

```yaml
# .gitlab-ci.yml (trechos)
stages: [test, release]

test:
  image: python:3.11
  script:
    - pip install -U pip
    - pip install -r requirements.txt
    - pip install .[dev]
    - pytest -q
  parallel:
    matrix:
      - PY: "3.9"
      - PY: "3.10"
      - PY: "3.11"
      - PY: "3.12"

release:
  stage: release
  image: python:3.11
  rules:
    - if: '$CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/'
  script:
    - pip install build twine
    - python -m build
    # opcional: publicar em registry interno do GitLab ou PyPI
    # - twine upload dist/*
  artifacts:
    paths: [dist/]
```

### 11.9 Sinalização no README

Badges:

```markdown
![status: stable](https://img.shields.io/badge/status-stable-blue)
![semver](https://img.shields.io/badge/semver-2.0.0-green)
![python](https://img.shields.io/badge/python-3.9--3.12-informational)
```

Bloco “Versão & Suporte”:

```markdown
## Versão & Suporte
- Versão atual: **0.1.0**
- Estado: **stable**
- Python: **3.9–3.12**
- Próxima remoção: _nenhuma_
```

### 11.10 Checklist de Versão

* [ ] `pyproject.toml` / `__version__` atualizados
* [ ] `CHANGELOG.md` com data e seções (Added/Changed/Deprecated/Fixed/Removed)
* [ ] README atualizado (estado, compatibilidade, remoções programadas)
* [ ] Matriz de CI verde (todas as versões de Python suportadas)
* [ ] *Warnings* de depreciação cobrindo caminhos antigos
* [ ] Tag anotada `vX.Y.Z` + Release Notes

---

## 12) Checklist de Entrega

* [ ] Estrutura de pastas conforme Seção 1
* [ ] *Type hints* e docstrings em 100% das APIs públicas
* [ ] Tratamento de erros + mensagens claras
* [ ] Logging configurado
* [ ] README completo (objetivo, escopo, uso, limitações, **versão & suporte**)
* [ ] `CHANGELOG.md` criado/atualizado
* [ ] `requirements.txt` fixado
* [ ] Notebook de demonstração cobrindo cada função
* [ ] Testes básicos com `pytest`
* [ ] Lint/format/type-check (Black/Ruff/Mypy)
* [ ] Versão inicial definida (ex.: `0.1.0`) e tag criada

---

## 13) Exemplos de Nomes

* **Módulo:** `converter.py`, `normalizer.py`, `validator.py`
* **Classe:** `DocumentConverter`, `CsvNormalizer`, `SchemaValidator`
* **Função:** `pdf_to_csv`, `pdf_to_html`, `csv_to_csv`
* **Variável:** `input_path`, `output_dir`, `page_range`, `overwrite`

---

## 14) Como Avaliar um Componente

* **Clareza** — docstrings + README com entradas/saídas inequívocas
* **Confiabilidade** — testes mínimos cobrem erros comuns
* **Reuso** — funções puras, dependências enxutas
* **Observabilidade** — logs úteis; erros com contexto
* **Limites** — README deixa claro o que *não* é suportado
* **Versionamento** — CHANGELOG, badges e depreciações bem sinalizadas

---