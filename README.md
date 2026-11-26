# RAG-ассистент «Фильмоскоп»

Проект содержит воспроизводимый Jupyter Notebook с RAG-системой для службы поддержки онлайн-кинотеатра «Фильмоскоп».

Система:

- очищает и анализирует базу знаний;
- строит чанки и эмбеддинги;
- использует FAISS/BM25/hybrid retrieval и reranking;
- генерирует ответы по найденному контексту;
- честно отказывается отвечать, если ответа нет в базе знаний;
- оценивает качество на 66 тестовых запросах, включая golden-context comparison и диагностику ошибок.

Главный артефакт проекта — `filmoskop_rag.ipynb` с сохранёнными outputs.

## Структура проекта

- `filmoskop_rag.ipynb` — основной notebook с полным RAG pipeline и оценкой качества.
- `filmoskop_kb.json` — база знаний и тестовые запросы.
- `spec.md` — спецификация задания и критерии готовности.
- `PLAN.md` — план доработок и статус выполненных шагов.
- `requirements.txt` — зависимости Python.
- `archive/` — архивные материалы предыдущих итераций.
- `temp/` — временные helper-скрипты и диагностические файлы, не являющиеся основным pipeline.

## Требования

- Python 3.11+.
- Windows PowerShell или совместимая shell-среда.
- Доступ к локальному OpenAI-compatible LLM endpoint для генерации и LLM-judge, если нужно заново выполнить все LLM-ячейки.
- Достаточно места для установки PyTorch, `sentence-transformers` и связанных ML-зависимостей.

## Установка

Из корня проекта:

```powershell
cd C:/PythonProg/ai-agent.rag-system2
python -m venv .venv
.venv/Scripts/Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Если PowerShell запрещает активацию virtualenv, можно выполнить:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.venv/Scripts/Activate.ps1
```

## Запуск notebook

```powershell
cd C:/PythonProg/ai-agent.rag-system2
.venv/Scripts/Activate.ps1
jupyter notebook filmoskop_rag.ipynb
```

После открытия notebook выполните ячейки сверху вниз.

Важно: полный evaluation на 66 запросах с generation, judge и golden-context comparison может выполняться долго, потому что делает много LLM-вызовов.

## Быстрая техническая проверка notebook

Проверить, что notebook читается как JSON, code-ячейки парсятся через AST и в сохранённых outputs нет error-ячeек:

```powershell
cd C:/PythonProg/ai-agent.rag-system2
.venv/Scripts/python.exe - <<'PY'
import ast
import json
from pathlib import Path

nb_path = Path('filmoskop_rag.ipynb')
nb = json.loads(nb_path.read_text(encoding='utf-8'))

errors = []
for idx, cell in enumerate(nb.get('cells', [])):
    if cell.get('cell_type') == 'code':
        try:
            ast.parse(''.join(cell.get('source', [])))
        except SyntaxError as exc:
            errors.append(f'AST error in cell {idx}: {exc}')
    for out_idx, output in enumerate(cell.get('outputs', []) or []):
        if output.get('output_type') == 'error':
            errors.append(f'Output error in cell {idx}, output {out_idx}: {output.get("ename")}: {output.get("evalue")}')

if errors:
    print('\n'.join(errors))
    raise SystemExit(1)

print(f'OK: {len(nb.get("cells", []))} notebook cells validated')
PY
```

## Итоговый статус качества

По последнему полному запуску:

- `retrieval_recall@5_non_gap = 0.9018` — целевой retrieval-порог достигнут;
- `correctness_retrieval_context = 0.8939` — немного ниже целевого порога `0.90`;
- `gap_exact_refusal_rate = 1.0` — все gap-запросы получили корректный отказ;
- основные зоны улучшения: `multi`-запросы и faithfulness генерации.
