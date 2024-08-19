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

## ✅ Como executar o projeto ✅
```
Comando 1
```
A execução é automatica, é iniciado a criação "startprocess" do processo quando cliente finaliza a compra de um ou mais produto(s) do venda-assistida da retaguarda RMS e via PDV Zanthus realiza o pagamento e acessa o portal de agendamentos de móveis e realiza o agendamento com o número do pedido e o cpf do cliente e selecionando uma data para o montador de móveis ser direcionado até seu respectivo endereço.

## 📌 (Montagem de Móveis) - Informações importantes sobre a aplicação 📌

Foram encontrados casos onde existem divergencia de informaçãoes no cadastro da retaguarda RMS, como também falta de classificação correta dos produtos que são disponiveis para montagem e não estão com flag igual "Sim" disponivel para montar, cliente que repassa informação incorreta como comprou em um CPF de terceiro e no agendamento informa seu CPF ou pedido incorreto.

## ⚠️ Problemas enfrentados ⚠️

Listo abaixo os problemas enfrentados mais comuns de acontecer no processo de Montagem de Móveis.

### Problema 1:
Descrição do problema:
Cliente entra em contato com a recepção da filial origem da compra relatando que não consegue agendar a montagem de móvel pois retorna o erro como se o produto não foi entregue ou cliente não comprou.
* Como solucionar: Deve-se consultar no banco de dados e confirmar se as informações inseridas estão corretas assim também se os itens retornados na coluna [DESCRICAO] são realmente para montagem e estão corretamente configurados na coluna [TIPO_1OU6] e [ITEM_MONTAVEL], somando também na verificação a coluna [EXISTE_RECEITA] igual a zero, [HORAS_MONTAGEM] e [QTDE_MONTADORES] diferentes de zero. Após analisar e corrigir estes pontos na consulta, pode solicitar para o cliente testar novamente, segue a query abaixo:
```
--CONSULTANDO PENDENCIAS:
select  
    (select PED_CGC_CPF from rms.ag3pvend@LK_RMS BX where BX.PED_NUM_PEDIDO_CP = PED_NUM_PEDIDO_DT) AS CPF,
    a.PED_NUM_PEDIDO_DT AS CODPEDIDO,
    a.PED_EAN_PRINC AS EAN,
    a.PED_PROD_PRINC_DT AS ITEM,
    b.GIT_DIGITO AS CODITEM_DIG,
    TRIM(git_descricao) AS DESCRICAO,
    b.GIT_TIPO_PRO AS TIPO_1OU6,
    b.GIT_TPO_EMB_VENDA AS EMB_VENDA,
    c.DET_ITEM_MONTAVEL AS ITEM_MONTAVEL,
    b.git_grupo AS GRUPO,
    b.git_subgrupo AS SUBGRUPO,
    b.git_categoria AS CATEGORIA,
    rms.get_rmscategoria@lk_rms(b.git_secao,git_grupo,git_subgrupo,git_categoria) AS FUNC_CATEGORIA_RMS,
    (SELECT COUNT(*) FROM rms.AA1CRECE@lk_rms, rms.AA3CITEM@lk_rms, rms.AA2CESTQ@lk_rms, rms.AA1DITEM@lk_rms, rms.AA1DRECE@lk_rms 
        WHERE GIT_COD_ITEM   = TRUNC(RECE_COMPONENTE/10) 
            AND GIT_DIGITO       = SUBSTR(TO_CHAR(RECE_COMPONENTE,'FM00000000'),8,1) 
            AND GET_COD_PRODUTO  = GIT_COD_ITEM || GIT_DIGITO
            AND DET_COD_ITEM     = GIT_COD_ITEM 
            AND DRECE_PRODUTO    = TRUNC(RECE_PRODUTO / 10) AND DRECE_COMPONENTE = TRUNC(RECE_COMPONENTE / 10)
            AND RECE_PRODUTO = a.PED_EAN_PRINC||a.PED_PROD_PRINC_DT ) AS EXISTE_RECEITA,
    (select HORAS_MONTAGEM from fluig.ti_montagem_agendamento_param p where p.codigo_grupo = b.git_grupo and p.codigo_subgrupo = b.git_subgrupo and p.CODIGO_CATEGORIA = b.git_categoria ) as HORAS_MONTAGEM,
    (select QTDE_MONTADORES from fluig.ti_montagem_agendamento_param p where p.codigo_grupo = b.git_grupo and p.codigo_subgrupo = b.git_subgrupo and p.CODIGO_CATEGORIA = b.git_categoria ) as QTDE_MONTADORES                
from rms.VW_AG3PVEDT@lk_rms a 
    inner join rms.aa3citem@lk_rms b on a.PED_PROD_PRINC_DT = b.GIT_COD_ITEM
        inner join rms.AA1DITEM@lk_rms c on c.DET_COD_ITEM = b.GIT_COD_ITEM
    where 1=1
    AND PED_NUM_PEDIDO_DT = 3160298;

--UPDATE CASO NECESSÁRIO:
update rms.AA1DITEM@lk_rms set DET_ITEM_MONTAVEL = 'S' where 1=1 and DET_COD_ITEM IN ( 135080 , 956606 , 956578 );

--TABELA DO FLUIG RESPONSÁVEL PELO CADASTRO DE HORAS E QUANTIDADE DE MONTADOR:
select * from fluig.ti_montagem_agendamento_param p where p.codigo_grupo = 10 and p.codigo_subgrupo = 5;

```

### Problema 2:
Descrição do problema:
Após inserir CPF e PEDIDO retorna nenhuma informação, existe uma consulta principal no dataset para buscar informações do cliente e item de faturamente do pedido, geração de nf na logistica o nome dele é: [ds_consulta_rms.js].
* Como solucionar: Deve-se revisar se a tarefa no banco que executa uma PROC da Visual MIX RMS está com o status BROKEN==Y, o job quando bloqueado não funciona e nem processas as NFe.

