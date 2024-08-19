# MONTAGEM DE MOVEIS.md
Nome do processo Diagrama: "montagem_moveis".

Nome do Portal Widget(Pública): "agendamentomontagem".

Desenvolvimento BPM de um diagrama de processo atrelado a um formulário HTML5 para acesso de cliente(s) que realizaram a compra de um ou mais produto(s) de montagem de móveis, produtos este que estão cadastrados na retaguarda RMS e ficam disponiveis para emissão do pedido (presencial ou e-commerce) de compra.

URL Pública: https://formosaweb.net.br/portal/1/agendamentomontagem

![image](https://github.com/user-attachments/assets/78af8cb3-1347-45e2-a972-3d3232c20df8)

![image](https://github.com/user-attachments/assets/8188c2c8-61c7-4641-af11-b929986d4451)

![image](https://github.com/user-attachments/assets/33b0a582-e044-4ff6-b473-fa605ebda4af)

![image](https://github.com/user-attachments/assets/ea1e5694-8a64-4663-a9fb-30ed3fe9f323)

### Tecnologias Utilizadas

* PL-SQL.
* JavaScript.
* Bibliotecas JS.
* Consulta pública via AJAX.

## Dependências e Versões Necessárias

* TOTVS Fluig Plataforma - 1.6+
* Web Service swagger
* moment.js
* datepicker.js
* backbone-min.js
* jquery.mask.js

## Como executar o projeto ✅
```
Comando 1
```
A execução é automatica, é iniciado a criação "startprocess" do processo quando cliente finaliza a compra de um ou mais produto(s) do venda-assistida da retaguarda RMS e via PDV Zanthus realiza o pagamento e acessa o portal de agendamentos de móveis e realiza o agendamento com o número do pedido e o cpf do cliente e selecionando uma data para o montador de móveis ser direcionado até seu respectivo endereço.

## 📌 (Montagem de Móveis) - Informações importantes sobre a aplicação 📌

Foram encontrados casos onde existem divergencia de informaçãoes no cadastro da retaguarda RMS, como também falta de classificação correta dos produtos que são disponiveis para montagem e não estão com flag igual "Sim" disponivel para montar, cliente que repassa informação incorreta como comprou em um CPF de terceiro e no agendamento informa seu CPF ou pedido incorreto.

## ⚠️ Problemas enfrentados

Listo abaixo os problemas enfrentados mais comuns de acontecer no processo de Montagem de Móveis.

### Problema 1:
Descrição do problema:
NFe de entradas para cortes das agendas [727,5,101] vem com o mesmo número de Nota que já foi usado anteriormente, não processa os cortes e deve ser confirmado se realmente a NFe de prateleira 730 e 580 recebimento não foi gerada no retarda mesmo que em datas futuras.
* Como solucionar: Este não é um "problema" no processo, e sim um problema operacional do fornecedor que gerou a NF com o mesmo número de outra NF usada anteriormente e também do recebimento do formosa/comprador que não força o mesmo a enviar número de notas corretas sem sequencias, deve-se acessar a tabela de controle NFe "fluig.ti_desossa_controle_nfe" e localizar a NFe de entrada pelo númeração que não processou, salvar as informações em alguma lugar seguro pois será usada futuramente, realizar o update na "fluig.ti_desossa_controle_nfe" na coluna "NOTA" alterar o número de NOTA para outro número próximo exemplo de:207551 para: 207551666 passando como filtro a filial, item, nota, dataNF e executar a desossa novamente, com isso programa de execução do fluig não vai mais achar este número de NF no not in da query de pendencias e executara/processará os cortes, após esta execução, deve-se voltar o número de nota exemplo realizando outro update de:207551666 para:207551.

### Problema 2:
Descrição do problema:
O usuários que estão responsáveis reclamam que a desossa não executou no dia anterior.
* Como solucionar: Deve-se revisar se a tarefa no banco que executa uma PROC da Visual MIX RMS está com o status BROKEN==Y, o job quando bloqueado não funciona e nem processas as NFe.

