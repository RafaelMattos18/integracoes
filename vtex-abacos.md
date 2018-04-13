# Integração VTex - Ábacos

---

## Produtos

### Processo

**Funcionalidade**: Consumir os produtos que estão no Ábacos, somente os faturados estarão disponiveis.

- Consulta produtos disponíveis no Ábacos (produtos que sofreram alterações)
- Ábacos nos envia uma lista de itens e transformamos em uma forma que consigamos trabalhar melhor. (mudanças de nomenclaturas)
- Diferenciação de produto ou SKU - baseada no Código pai - se codPai for 0 é um Produto - contrario SKU.
    - Pesquisa dentro da VTEX um produto, caso encontre é uma alteração senão é criação do produto.
    - Comportamento Alteração
        - Campos **Description, IsActive, IsVisible, Title, MetaTagDescription. LinkID** - nunca são alteradas, permanece do que está na Vtex.
        - **CategoryId** e **DepartmentID**(categoria Pai), **Name, BrandID** (marca), estes são passados na integração.
        - Caso não tenha uma categoria cadastrada no produto ele é inserido na categoria padrão - produtos sem categoria.
        - Passado o ID
    - Comportamento Criação
        - **Possui algo a mais**: Gera-se um LinkID (url do produto)
- Não informa o ID do produto
- Atualização do produto - envia dados para VTex
    - VTex retorna o produto e verificamos se está ativo
    - Caso não esteja ativo
        - Tenta ativar o produto
        - **Falha** - gravação de LOG
        
---

## Integração de SKU

- Consultar SKU dentro da VTEX
    - **ATUALIZAÇÃO SKU**
        - Conseguimos cadastrar somente quando há CodPai
        - IsActive e IsAvaiable nunca são alterados
        - Campo ModalID que controla estoque sempre 1
        - Passado o ID SKU
        - Atualização do SKU- envia dados para VTex
        - VTex retorna o SKU e verificamos se está ativo
            - Caso não esteja ativo
            - Tenta ativar o produto
            - Falha - gravação de LOG
        - SKU pode ou não ser Kit
        - Caso seja Kit
            - Percorre todos os componentes do KIT, consultando eles dentro da VTEX
            - Para cada componente do kit, monta um objeto, com ID SKU e ID Componente e Qtde do componente neste item.
            - Insere preço promocional dentro do KIT
                - Caso seja Zerado, assume o preço do item normal
        - Envia para VTEX
            - Cria vínculo entre SKU e componentes
    - **CRIAÇÃO SKU**
        - Código de barra é enviado somente na criação do produto
    - Envia um protocolo de consumação para o Ábacos, para o produto saia da fila.
    
---

## Status de Pedido

**Funcionalidade:** Consumir o status do pedido de mudança no Ábacos, somente os faturados - status disponiveis.

### Processo

- Consulta status disponíveis no Ábacos (pedidos que sofreram alterações de status - faturado)
-Localiza o pedido dentro da VTEX
    - Monta e envia um objeto de faturamento para VTEX
    - **Se recebido**
        - Confirma no Ábacos
        - Identifica se o pedido é DropShipping
            - Se for armazena em um banco de dados a parte.
    - **Se não recebido**
        - Não há confirmação no Ábacos
        - Roda a CRON novamente para requisitar
- Envia um protocolo de consumação para o Ábacos, de status do pedido para que saia da fila.

---

## Marcas e categoria

**Funcionalidade:** Consome as marcas disponiveis no Ábacos (sofreu alteração)

### Processo
- Envia marca para VTEX
    - Caso a VTEX receba
        - Confirmação no Ábacos
    - Caso não recebe
        - Roda a CRON novamente para requisitar

---
        
## Regras

### Prazos de entrega

**Se o canal de venda for B2W**
- Pega a data de criação destes pedido, soma-se mais 8 dias VTex (corridos) e mais dias que a VTex nos passa da transportadora.

**Se não for canal B2W**
- Data criação + Data Transportadora VTex.
