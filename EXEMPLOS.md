# Exemplos

Exemplos de código do mundo real demonstrando os quatro princípios. Cada exemplo mostra o que LLMs comumente fazem de errado e como corrigir.

---

## 1. Pense Antes de Codificar

### Exemplo 1: Suposições Ocultas

**Solicitação do Usuário:** "Adicione uma funcionalidade para exportar dados do usuário"

**❌ O Que LLMs Fazem (Suposições Erradas)**

```python
def export_users(format='json'):
    """Export all users to a file."""
    users = User.query.all()
    
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])
    
    return f"Exported {len(users)} users"
```

**Problemas:**
- Assumiu que deveria exportar TODOS os usuários (e quanto à paginação? privacidade?)
- Assumiu local do arquivo sem perguntar
- Assumiu quais campos incluir
- Assumiu nomes de colunas CSV sem verificar a estrutura real dos dados

**✅ O Que Deveria Acontecer (Expor Suposições)**

```
Antes de implementar, preciso esclarecer:

1. **Escopo**: Exportar todos os usuários ou subconjunto filtrado? (implicações de privacidade)
2. **Formato**: Você mencionou "exportar" — você quis dizer:
   - Download de arquivo no navegador?
   - Job em background com notificação por e-mail?
   - Endpoint de API retornando dados?
3. **Campos**: Quais campos do usuário? (alguns podem ser sensíveis)
4. **Volume**: Quantos usuários tipicamente? (afeta a abordagem)

Abordagem mais simples: Adicionar um endpoint de API que retorna JSON paginado.
Precisaria de mais informações para exportações baseadas em arquivo.

Qual é sua preferência?
```

### Exemplo 2: Múltiplas Interpretações

**Solicitação do Usuário:** "Torne a busca mais rápida"

**❌ O Que LLMs Fazem (Escolhem Silenciosamente)**

```python
# Adds caching, database indexes, and async processing
# without asking which "faster" matters
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 lines of optimization code
```

**✅ O Que Deveria Acontecer**

```
"Torne a busca mais rápida" pode significar:

1. **Tempo de resposta mais rápido** (< 100ms em vez de 500ms)
   - Adicionar índices no banco de dados
   - Cachear consultas frequentes
   - Esforço estimado: 2 horas

2. **Lidar com mais buscas concorrentes** (throughput)
   - Adicionar async/await
   - Pool de conexões
   - Esforço estimado: 4 horas

3. **Velocidade percebida mais rápida** (UX)
   - Mostrar resultados parciais imediatamente
   - Carregamento progressivo
   - Esforço estimado: 3 horas

A busca atual leva ~500ms para consultas típicas.
Qual aspecto importa mais para o seu caso de uso?
```

---

## 2. Simplicidade Primeiro

### Exemplo 1: Super-abstração

**Solicitação do Usuário:** "Adicione uma função para calcular desconto"

**❌ O Que LLMs Fazem (Superengenharia)**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount
    
    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config
    
    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# Usage requires 30+ lines of setup for a simple calculation
```

**✅ O Que Deveria Acontecer (Simples)**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """Calculate discount amount. percent should be 0-100."""
    return amount * (percent / 100)

# Usage
discount = calculate_discount(100.0, 10.0)  # $10 off
```

**Quando adicionar complexidade:** Apenas quando você realmente precisar de múltiplos tipos de desconto. Se esse requisito surgir depois, refatore então.

### Exemplo 2: Funcionalidades Especulativas

**Solicitação do Usuário:** "Salve preferências do usuário no banco de dados"

**❌ O Que LLMs Fazem (Funcionalidades Desnecessárias)**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()
    
    def save(self, user_id: int, prefs: dict, 
             merge: bool = True, 
             validate: bool = True,
             notify: bool = False) -> bool:
        """Save preferences with optional merging, validation, and notifications."""
        
        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)
        
        existing = self.get(user_id) if merge else {}
        
        if merge:
            existing.update(prefs)
            prefs = existing
        
        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )
        
        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)
        
        if notify:
            self.notify_preference_change(user_id, prefs)
        
        return True
    
    def notify_preference_change(self, user_id: int, prefs: dict):
        # Another 30 lines for a feature nobody asked for
        pass
```

**✅ O Que Deveria Acontecer (Apenas o Solicitado)**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """Save user preferences to database."""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**Adicionar depois se necessário:** Cache (quando performance importar), validação (quando dados ruins aparecerem), merge (quando o requisito surgir).

---

## 3. Mudanças Cirúrgicas

### Exemplo 1: Refatoração Incidental

**Solicitação do Usuário:** "Corrija o bug onde e-mails vazios quebram o validador"

**❌ O Que LLMs Fazem (Demais)**

```diff
  def validate_user(user_data):
-     # Check email format
-     if not user_data.get('email'):
+     """Validate user data."""
+     email = user_data.get('email', '').strip()
+     
+     # Validate email
+     if not email:
          raise ValueError("Email required")
-     
-     # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
          raise ValueError("Invalid email")
      
-     # Check username
-     if not user_data.get('username'):
+     # Validate username
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Username required")
+     if len(username) < 3:
+         raise ValueError("Username too short")
+     if not username.isalnum():
+         raise ValueError("Username must be alphanumeric")
      
      return True
```

**Problemas:**
- "Melhorou" validação de e-mail além da correção do bug
- Adicionou validação de username que ninguém pediu
- Alterou comentários
- Adicionou docstring

**✅ O Que Deveria Acontecer (Cirúrgico)**

```diff
  def validate_user(user_data):
      # Check email format
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("Email required")
      
      # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email:
          raise ValueError("Invalid email")
      
      # Check username
      if not user_data.get('username'):
          raise ValueError("Username required")
      
      return True
```

**Alterou apenas:** As linhas específicas que corrigem o tratamento de e-mail vazio.

### Exemplo 2: Deriva de Estilo

**Solicitação do Usuário:** "Adicione logging à função de upload"

**❌ O Que LLMs Fazem (Reformatam Tudo)**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """Upload file to destination with logging."""
+     logger.info(f"Uploading {file_path} to {destination}")
+     
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-         
-         response = requests.post(destination, files={'file': data})
-         
-         if response.status_code == 200:
-             return True
-         else:
-             return False
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
+         success = response.status_code == 200
+         
+         if success:
+             logger.info(f"Upload successful: {file_path}")
+         else:
+             logger.error(f"Upload failed: {file_path}, status={response.status_code}")
+         
+         return success
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f"Upload error for {file_path}: {e}")
          return False
```

**Problemas:**
- Mudou estilo de aspas ('' para "")
- Adicionou type hints que ninguém pediu
- Adicionou docstring
- Reformatou espaçamento
- Alterou lógica de retorno booleano

**✅ O Que Deveria Acontecer (Combinar com Estilo Existente)**

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
  def upload_file(file_path, destination):
+     logger.info(f'Starting upload: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()
          
          response = requests.post(destination, files={'file': data})
          
          if response.status_code == 200:
+             logger.info(f'Upload successful: {file_path}')
              return True
          else:
+             logger.error(f'Upload failed: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f'Upload error: {file_path}')
          return False
```

**Combinou com:** Aspas simples, sem type hints, padrão booleano existente, estilo de espaçamento.

---

## 4. Execução Orientada a Objetivos

### Exemplo 1: Vago vs. Verificável

**Solicitação do Usuário:** "Corrija o sistema de autenticação"

**❌ O Que LLMs Fazem (Abordagem Vaga)**

```
Vou corrigir o sistema de autenticação:
1. Revisando o código
2. Identificando problemas
3. Fazendo melhorias
4. Testando as mudanças

[Prossegue fazendo mudanças sem critérios claros de sucesso]
```

**✅ O Que Deveria Acontecer (Objetivos Verificáveis)**

```
Para "corrigir autenticação", preciso definir critérios de sucesso.
Qual problema específico estamos resolvendo?

Se o problema é "usuários permanecem logados após troca de senha":

Plano:
1. Escrever teste: Trocar senha → verificar sessão antiga invalidada
   Verificar: Teste falha (reproduz o bug)

2. Implementar: Invalidar sessões na troca de senha
   Verificar: Teste passa

3. Verificar casos extremos: Múltiplas sessões ativas, mudanças concorrentes
   Verificar: Testes adicionais passam

4. Verificar sem regressão: Testes de auth existentes ainda passam
   Verificar: Suite completa verde

Cobertura atual de testes para auth: [mostra cobertura]
Qual é o problema específico de auth que você está vendo?
```

### Exemplo 2: Multi-etapas com Verificação

**Solicitação do Usuário:** "Adicione rate limiting à API"

**❌ O Que LLMs Fazem (Tudo de Uma Vez)**

```python
# Implements full rate limiting with Redis, multiple strategies,
# configuration system, and monitoring in one 300-line commit
# No clear verification steps
```

**✅ O Que Deveria Acontecer (Incremental com Verificação)**

```
Plano para rate limiting:

1. Adicionar rate limiting básico em memória (endpoint único)
   Verificar: 
   - Teste: 100 requisições → primeiras 10 sucedem, resto recebe 429
   - Manual: curl endpoint 11 vezes, ver erro de rate limit

2. Extrair para middleware (aplicar a todos os endpoints)
   Verificar:
   - Teste: Rate limits aplicam a /users e /posts
   - Testes de endpoints existentes ainda passam

3. Adicionar backend Redis (para multi-servidor)
   Verificar:
   - Teste: Rate limit persiste após reinícios do app
   - Teste: Duas instâncias compartilham contador de rate limit

4. Adicionar configuração (taxas por endpoint)
   Verificar:
   - Teste: /search permite 10/min, /users permite 100/min
   - Arquivo de config parseado corretamente

Cada etapa é independentemente verificável e implantável.
Começar com a etapa 1?
```

### Exemplo 3: Verificação Test-First

**Solicitação do Usuário:** "A ordenação quebra quando há pontuações duplicadas"

**❌ O Que LLMs Fazem (Corrigem Sem Reproduzir)**

```python
# Immediately changes sort logic without confirming the bug
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ O Que Deveria Acontecer (Reproduzir Primeiro)**

```python
# 1. First, write a test that reproduces the issue
def test_sort_with_duplicate_scores():
    """Test sorting when multiple items have same score."""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]
    
    result = sort_scores(scores)
    
    # The bug: order is non-deterministic for duplicates
    # Run this test multiple times, it should be consistent
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# Verify: Run test 10 times → fails with inconsistent ordering

# 2. Now fix with stable sort
def sort_scores(scores):
    """Sort by score descending, then name ascending for ties."""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# Verify: Test passes consistently
```

---

## Resumo de Anti-padrões

| Princípio | Anti-padrão | Correção |
|-----------|-------------|----------|
| Pense Antes de Codificar | Assume silenciosamente formato de arquivo, campos, escopo | Liste suposições explicitamente, peça esclarecimento |
| Simplicidade Primeiro | Strategy pattern para cálculo de desconto único | Uma função até a complexidade ser realmente necessária |
| Mudanças Cirúrgicas | Reformata aspas, adiciona type hints ao corrigir bug | Altere apenas linhas que corrigem o problema reportado |
| Execução Orientada a Objetivos | "Vou revisar e melhorar o código" | "Escrever teste para bug X → fazê-lo passar → verificar sem regressões" |

## Insight Principal

Os exemplos "supercomplicados" não são obviamente errados — seguem padrões de design e boas práticas. O problema é o **timing**: adicionam complexidade antes de ser necessária, o que:

- Torna o código mais difícil de entender
- Introduz mais bugs
- Leva mais tempo para implementar
- É mais difícil de testar

As versões "simples" são:
- Mais fáceis de entender
- Mais rápidas de implementar
- Mais fáceis de testar
- Podem ser refatoradas depois quando a complexidade for realmente necessária

**Bom código é código que resolve o problema de hoje de forma simples, não o problema de amanhã prematuramente.**
