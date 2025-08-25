# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**
Há diferença, pois o printf é capaz de escrever textos para o usuário, formatar dados e é utilizado para casos que não seja necessário uma performance crítica.
E o write é capaz de escrever, além de texto, dados binários, controlar quando enviar os dados e possui um comportamento previsível.

**3. Qual método é mais previsível? Por quê você acha isso?**

O write é mais previsível porque é uma chamada de sistema direta: você especifica o que escrever e o tamanho, e o kernel executa exatamente isso sem camadas extras nem buffers automáticos.

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

O file descriptor usado foi o 3. O 0, 1 e 2 sao reservados pelo sistema para stdin, stdout e stderr, respectivamente.

**2. Como você sabe que o arquivo foi lido completamente?**

Sabemos que o arquivo foi lido completamente pois a ultima saida de read é 0, ou seja, não existem mais dados para serem lidos.

**3. Por que verificar retorno de cada syscall?**

Verificamos o retorno de cada syscall para garantir que nao havera erros durante a leitura.


## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read():21
- Tempo: 0,000084 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |       82        | 0,001612  |
| 64          |       21        | 0,000084  |
| 256         |       06        | 0,000069  |
| 1024        |       02        | 0,000061  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

Quanto maior o numero do buffer, são necessarias menos chamadas de read, reduzindo o tempo de leitura.

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

Não, nem todas as chamadas read() retornam BUFFER_SIZE bytes. Normalmente retornam BUFFER_SIZE, exceto a última chamada, que retorna apenas os bytes restantes até o fim do arquivo.

**3. Qual é a relação entre syscalls e performance?**

Quanto menos syscalls (read, nesse caso) forem necessárias, maior é a performance. Vemos isso aumentando a quantidade do BUFFER_SIZE.

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0,000233 segundos
- Throughput: 5716,87 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

Verificar bytes_escritos == bytes_lidos garante que todos os dados foram realmente gravados e evita perda de informações.

**2. Que flags são essenciais no open() do destino?**

As flags essenciais sao: O_WRONLY, O_CREAT, O_TRUNC: escrever, criar se não existir e apagar o conteúdo se existir.

**3. O número de reads e writes é igual? Por quê?**

Geralmente são iguais, porque cada read() gera um write(), mas podem ser diferentes se a última leitura retornar menos bytes que o buffer

**4. Como você saberia se o disco ficou cheio?**

Você saberia porque a chamada write() retornaria -1 e errno seria configurado para ENOSPC, indicando que não há espaço suficiente no disco.

**5. O que acontece se esquecer de fechar os arquivos?**

Se não fechar os arquivos, dados podem não ser gravados e os descritores ficam abertos, consumindo recursos e podendo causar erros.

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

As syscalls mostram a transição porque, ao serem chamadas, o processo passa do espaço de usuário para o espaço de kernel, onde o sistema operacional executa a operação.

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

File descriptors são essenciais para acessar arquivos, sockets e dispositivos, funcionando como ponte entre usuário e kernel e permitindo controlar leitura, escrita e fechamento de forma segura.

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

Buffer pequeno gera muitas syscalls e diminui performance; buffer grande reduz chamadas e aumenta throughput, mas tamanho excessivo pode desperdiçar memória.

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** O programa

**Por que você acha que foi mais rápido?**

O programa faz somente a cópia essencial, sem verificações extras ou operações adicionais que o cp pode executar

---

## 📤 Entrega
Certifique-se de ter:
- [OK] Todos os códigos com TODOs completados
- [OK] Traces salvos em `traces/`
- [OK] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
