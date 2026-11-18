# Relatório de Engenharia e Auditoria de Software CLP
## Projeto: Aula 02 – Campainha Residencial (Framework CLP Branqs)

**Desenvolvedor:** Matheus Silva  
**Ferramentas:** Editor Programa CLP (Open Watcom IDE), LibreOffice Draw (Branqs Extension), Simulador DOSShell/DOSBox, Framework CLP Branqs v02r03 (bcv03r88)  
**Auditoria & Apoio:** Antigravity 

---

## 1. Objetivo do Teste de Estágio
Desenvolver o software de controle e a interface gráfica (IHM) para o sistema de campainha de uma residência automatizada, conforme as diretrizes do **Módulo I – Desenvolvimento de Software CLP Básico (Aula 02)** da Branqs Automação.

### Requisitos Funcionais:
1. **Acionamento Direto:** A campainha física deve ser acionada imediatamente quando o interruptor for pressionado e desacionada assim que for liberado.
2. **Supervisão Visual na IHM:** O status da entrada (interruptor) e da saída (campainha) deve ser apresentado em tempo real na tela de **Processo** da IHM.
3. **Deploy Industrial:** O programa deve ser compilado em binário de 32 bits (CauseWay DOS) e empacotado para instalação em CLP industrial real via pendrive.

---

## 2. Mapeamento de Entradas e Saídas (`DECLARA.CLP`)
A relação com os bornes físicos das placas de expansão (BC8E para entradas e BC8S para saídas) é estabelecida pela ordem sequencial de declaração.

```clp
////////////////////////////////////////////////////////////////////////////////////////////////
// Declaracao Entradas digitais
////////////////////////////////////////////////////////////////////////////////////////////////
CAMPO_ENTRADA_DIGITAL(entradaInterruptor)

////////////////////////////////////////////////////////////////////////////////////////////////
// Declaracao Saidas digitais
////////////////////////////////////////////////////////////////////////////////////////////////
CAMPO_SAIDA_DIGITAL(saidaCampainha)
```

- `entradaInterruptor`: Vinculada à Entrada 1 da primeira placa de entradas digitais (BC8E).
- `saidaCampainha`: Vinculada à Saída 1 da primeira placa de saídas digitais (BC8S).

---

## 3. Lógica de Controle (`LOGICA.CLP`)
A lógica utiliza a linguagem estruturada/textual do Framework Branqs, operando sobre o registrador **ELC** (*Estado Lógico Corrente*), equivalente a um rung em diagrama Ladder.

```clp
////////////////////////////////////////////////////////////////////////////////////////////////
//Comando
//////////////////////////////////////////////////////////////////////////////////////////////// 
SEL (entradaInterruptor)
MEMO (saidaCampainha)
```

### Análise Lógica:
- `SEL (entradaInterruptor)`: Instrução *Se Ligado*. Carrega o estado lógico da variável de entrada no acumulador ELC (equivalente a um contato normalmente aberto).
- `MEMO (saidaCampainha)`: Instrução *Memoriza*. Atribui o valor acumulado no ELC à bobina da saída física da campainha.

---

## 4. Interface Homem-Máquina (`PAGINAS\processo.cmp` e `.odg`)
Na tela `processo`, foram configurados os objetos de monitoramento visual:

| Campo / Tag | Tipo | Edição | Nível Senha | Símbolo Falso | Símbolo Verdadeiro | Descrição |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `entradaInterruptor` | `bool` | 0 (Não editável)* | 0 | 0 | 1 | Status do interruptor de entrada |
| `saidaCampainha` | `bool` | 0 (Não editável) | 0 | 0 | 1 | Status do acionamento da campainha |

*\*Nota: No modo simulação (`-s`), campos `bool` mapeados para entradas permitem alternância via teclado (`Tab` + `Enter`) para fins de teste de bancada.*

Sintaxe compilada no descritor de tela (`processo.cmp`):
```text
11,8,bool,entradaInterruptor,0,0,0,1
11,13,bool,saidaCampainha,0,0,0,1
```

---

## 5. Auditoria de Integridade e Correções Realizadas
Durante a auditoria socrática da base de código inicial, foram identificados e saneados os seguintes pontos de dívida técnica:

1. **Ausência da Stack de Runtime:** O diretório continha apenas arquivos isolados sem o núcleo `CODBASE.CPP`, as bibliotecas de cabeçalho (`clp.h`) e o runtime `bcv03r88.lib`. Toda a árvore industrial do Framework Branqs v02r03 foi reconstituída e integrada.
2. **Padronização Semântica:** A nomenclatura genérica (`E1`/`S4`) foi substituída pelos identificadores canônicos exigidos na especificação da Aula 02 (`entradaInterruptor` e `saidaCampainha`).
3. **Eliminação de Ruído de Compilação:** Removidos caracteres literais inválidos (`...`) no bloco `MODO DE OPERACAO` que geravam erro fatal de análise sintática no pré-processador C++.
4. **Alinhamento IHM:** Criação dos campos de exibição booleana nos arquivos de tela (`definicaoLayoutPaginas.odg` e `processo.cmp`).

---

## 6. Procedimento de Teste e Validação

### A. Simulação em Bancada (PC)
1. Abrir o atalho **Simulador** (DOSShell).
2. Selecionar a entrada `aula 02` (executável: `progCLP.exe`, parâmetro: `-s`).
3. Executar o simulador.
4. Pressionar `F2` para navegar até a página **Processo**.
5. Posicionar o cursor sobre o campo do interruptor, pressionar `Tab` (alterna para 1) e `Enter`.
6. **Evidência de Sucesso:** O campo da campainha transiciona imediatamente de `0` para `1`.
7. Pressionar `Tab` novamente (alterna para 0) e `Enter`. O campo da campainha retorna imediatamente para `0`.

### B. Deploy e Teste em CLP Real
1. Executar `deploy_v02_r03.bat` para gerar o pacote `progCLP.zip`.
2. Copiar o arquivo `progCLP.zip` para a pasta `/branqs/` na raiz de um pendrive formatado em FAT32.
3. Conectar o pendrive na porta USB do CLP Branqs e reiniciar a alimentação.
4. No menu de boot da IHM, pressionar `1` para gravar o software na memória não-volátil.
5. Pressionar `F2` na IHM e testar o interruptor físico na placa BC8E.

---

**Veredito:** Projeto 100% aderente às normas técnicas e especificações da Branqs Automação. Evidência comprovada.
