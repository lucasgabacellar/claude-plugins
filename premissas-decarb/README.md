# Premissas Decarb

Plugin para o Claude Cowork que gerencia o banco de premissas de descarbonização da PSR.

## Skills

### extract-assumptions
Extrai premissas de documentos (PDF, Word, Excel) ou do chat, valida com o usuário via interface interativa, e insere no workbook Premissas_Decarb.

**Triggers**: "extrair premissas", "extract assumptions", "nova premissa", "importar premissas", ou quando um arquivo com premissas é anexado.

### query-assumptions
Consulta premissas existentes no workbook. Busca por texto livre em qualquer coluna.

**Triggers**: "buscar premissa", "qual premissa", "temos premissa de", "what assumptions".

## Requisitos

- Workbook `Premissas_Decarb.xlsm` acessível na pasta conectada
- Python com openpyxl (para leitura/escrita do workbook)

## Segurança

O plugin NUNCA modifica a estrutura do workbook, VBA, shapes ou proteção. Apenas adiciona linhas a tabelas existentes, respeitando as mesmas regras de um usuário manual.
