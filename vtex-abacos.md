# Produtos

## Processo

**Funcionalidade: ** Consumir os produtos que estão no Ábacos, somente os faturados estarão disponíveis.

* Consulta produtos disponíveis no Ábacos (produtos que sofreram alterações)
* Ábacos nos envia uma lista de itens e transformamos em uma forma que consigamos trabalhar melhor. (Mudanças de nomenclaturas)
* Diferenciação de produto ou SKU - baseada no Código pai - se codPai for 0 é um Produto - contrario SKU.
	* Pesquisa dentro da VTEX um produto, caso encontre é uma alteração senão é criação do produto.
	* Comportamento Alteração
		* Campos Description, IsActive, IsVisible, Title, MetaTagDescription. LinkID - nunca são alteradas, permanece do que está na Vtex.
		* CategoryId e DepartmentID(categoria Pai), Name, BrandID (marca), estes são passados na integração.
		* Caso não tenha uma categoria cadastrada no produto ele é inserido na categoria padrão - produtos sem categoria.
		* Passado o ID
	* Comportamento Criação
		* Possui algo a mais:
		* Gera-se um LinkID (url do produto)
* Não informa o ID do produto
* Atualização do produto - envia dados para VTex
	* VTex retorna o produto e verificamos se está ativo
	* Caso não esteja ativo
		* Tenta ativar o produto
		* Falha - gravação de LOG
---

# Integração de SKU

* Consultar SKU dentro da VTEX
	* Atualização SKU
		* Conseguimos cadastrar somente quando há CodPai
		* IsActive e IsAvaiable nunca são alterados
		* Campo ModalID que controla estoque sempre 1
		* Passado o ID SKU
		* Atualização do SKU- envia dados para VTex
		* VTex retorna o SKU e verificamos se está ativo
			* Caso não esteja ativo
			* Tenta ativar o produto
			* Falha - gravação de LOG
		* SKU pode ou não ser Kit
			* Caso seja Kit
				* Percorre todos os componentes do KIT, consultando eles dentro da VTEX
				* Para cada componente do kit, monta um objeto, com ID SKU e ID Componente e Qtde do componente neste item.
				* Insere preço promocional dentro do KIT
					* Caso seja zerado, assume o preço do item normal
		* Envia para VTEX
			* Cria vínculo entre SKU e componentes
	* Criação SKU
		* Código de barra é enviado somente na criação do produto
* Envia um protocolo de consumação para o Ábacos, para o produto saia da fila.

---

# Status de Pedido

**Funcionalidade:** Consumir o status do pedido de mudança no Ábacos, somente os faturados - status disponíveis.

## Processo

* Consulta status disponíveis no Ábacos (pedidos que sofreram alterações de status - faturado)
* Localiza o pedido dentro da VTEX
	* Monta e envia um objeto de faturamento para VTEX
	* Se recebido
		* Confirma no Ábacos
		* Identifica se o pedido é DropShipping
			* Se for armazena em um banco de dados a parte.
    * Se não recebido
    	* Não há confirmação no Ábacos
    	* Roda a CRON novamente para requisitar 

* Envia um protocolo de consumação para o Ábacos, de status do pedido para que saia da fila.

---

# Marcas e categoria

**Funcionalidade:** Consome as marcas disponíveis no Ábacos (sofreu alteração)

## Processo

* Envia marca para VTEX
	* Caso a VTEX receba
		* Confirmação no Ábacos
    * Caso não recebe
    	* Roda a CRON novamente para requisitar

---

# Estoque

**Funcionalidade:** Consulta estoque disponível do banco de replicação de estoque.

## Processo

* Há um banco que possui uma lógica DE->. Para que ao procuramos um DK encontra-se o seu SKU.

* **Caso encontre**
	* gera um modelo no padrão JSON e envia para VTEX.
	```
	[
  		{
    		"wareHouseId": "1937054",
    		"itemId": "2390059",
    		"unlimitedQuantity": false,
    		"dateUtcOnBalanceSystem": "2016-15-04T17:00:00.4630000+00:00",
    		"quantity": 100
  		}
	]```
	
	* Atualiza o estoque na VTEX
	* Confirmação de estoque consumido

* **Caso não encontre**
	* Procura na VTEX nosso SKU com referência o campo RefID
	* Atualiza o estoque na VTEX
	* Atualiza o nosso banco
	* Confirmação de estoque consumido

### Serviços 

#### Inserção banco de estoque

* vai nos ábacos consulta o estoque disponível (produtos que sofreram alteração de estoque) e atualiza no banco
* confirma no Ábacos
> **Obs.: ** Foi feito este processo intermediado com o nosso banco, pois havia registros desnecessário e duplicados no Ábacos. Neste modelo inserimos no nosso banco e somente atualizamos no Ábacos.

#### Consulta estoque disponível 

* Este serviço consulta o banco que foi populado

---

# Integração de Preços

**Funcionalidade:** Consulta os preços disponíveis no Ábacos (que sofreram alterações).

## Processo

* Verifica os preços DE e preço POR se é maior que 0
	* Verifico se produto ou SKU
		* Se produto
			* Confirma no Ábacos
		* Se SKU
			* Atualiza preço na VTEX
			* Confirma no Ábacos

* Fora esta condição gera-se um Log

---

# Integração de Pedido

**Funcionalidade:** Integrar pedido dentro do ábacos

## Processo

* Buscar pedido dentro da VTEX que possui este status "**ready-for-handling**".

**Tabela de Tipo de Status**

| OMS                                   | Meus Pedidos                              | Gateway     | API Rest                                           | WebService |
|---------------------------------------|-------------------------------------------|-------------|----------------------------------------------------|------------|
| Aguardando confirmaзгo do Seller      | Processando Pagamento                     | Authorizing | waiting-for-seller-confirmation                    | AAP        |
| Pagamento Pendente                    | Processando Pagamento                     | Authorizing | payment-pending                                    | _NEAS      |
| Pagamento Aprovado                    | Preparando Entrega                        | Approved    | payment-approved                                   | AAP        |
| Pagamento Negado                      | Pagamento Negado                          | Cancelled   | Payment-denied                                     | CAN        |
| Aguardando decisгo do Seller          | Verificando possibilidade de cancelamento | Approved    | waiting-for-seller-decision                        | AAP        |
| Aguardando autorizaзгo para despachar | Preparando Entrega                        | Approved    | waiting-ffmt-authorization / authorize-fulfillment | ERP        |
| Carкncia para Cancelamento            | Preparando Entrega                        | Approved    | window-to-cancel                                   | ERP        |
| Pronto para o Manuseio                | Preparando Entrega                        | Approved    | ready-for-handling                                 | CAP        |
| Iniciar Manuseio                      | Preparando Entrega                        | Approved    | start-handling                                     | CAP        |
| Preparando Entrega                    | Entregando Produtos                       | Approved    | ship / invoice                                     | ERP        |
| Cancelamento Solicitado               | Cancelado                                 | Approved    | request-cancel                                     | CAN        |
| Verificando Envio                     | Processando Pedido                        | Approved    | order-accepted                                     | ERP        |
| Faturado                              | Enviado                                   | Finished    | shipped / invoiced                                 | ETR        |
| Cancelar                              | Processando Cancelamento                  | Cancelled   | cancel                                             | CAN        |
| Cancelado                             | Cancelado                                 | Cancelled   | canceled                                           | CAN        |

- Veriifica a existencia na Abacos
	- **Se existir**
		- Atualiza na VTEX para "**start-handling**"
	- **Se não existir**
		* Faz uma lista de clientes baseados em pedidos e insere ou atualiza no Ábacos
		* Verifica se o pedido contém item de DROPSHIPPING
		* **Se existir**
			* Executa um tratamento nos pedidos que é: 
				* Identificar Canal e SubCanal de Vendas.
				* Valores de Encargos
					$$ valorPago - (valorItens + valorFrete - valorDesconto);$$
			* Executa uma criptografia no campo de e-mail, CPF e CNPJ
			* Para cada fornecedor diferente no pedido irá gerar um novo pedido, alterando:
				* número do pedido para numeroDoPedido-Sigla (Sigla inserida junto com o cadastro do fornecedor).
				* replica a forma de pagamento original, alterando os valores (valorItens + valorFrete = valorPedido).
				* armazena este pedido no banco DrpoShipping e insere no Àbacos.

		* **Se não existir**
			* Executa um tratamento nos pedidos que é: 
				* Identificar Canal e SubCanal de Vendas.
				* Valores de Encargos
					$$ valorPago - (valorItens + valorFrete - valorDesconto); $$
			* Executa uma criptogrtafia no campo de e-mail, CPF e CNPJ

---

# Regras

## Prazos de entrega

- **Se o canal de venda for B2W**
	- Pega a data de criação destes pedido, soma-se mais 8 dias VTex (corridos) e mais dias que a VTex nos passa da transportadora.

- **Se não for canal B2W**
	- Data criação + Data Transportadora VTex.
