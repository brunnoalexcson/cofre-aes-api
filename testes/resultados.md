# Resultados dos testes

## Teste 1 — Nonces distintos

### Procedimento

Cadastre duas vezes a mesma senha, com títulos diferentes, no mesmo cofre. Consulte a tabela segredos no Supabase.

### Resultado esperado

Os campos nonce e criptograma são diferentes nos dois registros, ainda que a senha protegida seja idêntica.

### Resultado observado
Foram adicionadas duas senhas iguais ao cofre com títulos diferentes e foram gerados nonce e criptograma diferentes no banco de dadoscomo esperado.
No endpoint de listar os segredos os dois retornam como é observado na captura de tela.

### Status

Aprovado

## Teste 2 — Senha-mestra incorreta

### Procedimento

Solicite a leitura de um segredo informando uma senha-mestra errada.

### Resultado esperado

Resposta 401, sem qualquer conteúdo do segredo no corpo da resposta.

### Resultado observado

Tentei listar os segredos de um cofre com senha errada e retornou o erro 401 Unauthorized.

### Status

Aprovado
---

## Teste 3 — O que o invasor enxerga

### Procedimento

No Supabase, execute uma consulta direta à tabela e registre o resultado:

    select titulo, usuario, nonce, criptograma, etiqueta
    from public.segredos;

### Resultado esperado

Nenhuma senha legível. Apenas cadeias em Base64 sem significado aparente.

### Resultado observado

Rodei a consulta sql e não retornou nenhuma senha elegível.

### Status

Aprovado

---

## Teste 4 — Registro adulterado

### Procedimento

### Resultado esperado

### Resultado observado

### Status


---

## Teste 5 — Troca de criptogramas entre registros

### Procedimento

### Resultado esperado

### Resultado observado

### Status
