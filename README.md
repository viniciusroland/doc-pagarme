# Doc sobre pagarme!

A pagarme é uma plataforma de pagamento online brasileira. Foi fundada por 2 moleques de 18 anos, foda né bixo ¯\_(ツ)_/¯

A API funciona basicamente com requisições http (GET, POST, ETC) usando a biblioteca deles que tem suporte para algumas linguagens, como python e javascript.

Geralmente a gente usa uma plataforma de pagamento para ela lidar com os dados sensíveis de cartão de crédito, fazer toda a segurança a gente só se preocupar mesmo com o desenvolvimento da nossa aplicação.


# API

site oficial - https://docs.pagar.me/reference#principios-basicos
faq - https://pagarme.zendesk.com/hc/pt-br/sections/200093186-D%C3%BAvidas-Frequentes

## Criando uma conta
Acesse : https://dashboard.pagar.me/#/login e vá no cadastro de desenvolvedor abaixo do botão de login.

Crie sua conta, logue, e vá em: 
'Minha conta' -> 'Configurações' -> 'Api keys'

Guarde essas keys, é com elas que você irá se autenticar no back-end.
A primeira key é a principal, a outra é para casos mais específicos.

## Autenticação

```python
import pagarme

pagarme.authentication_key('SUA_API_KEY')
# ex de api_key 'ak_test_uyreegauenj2VuJ46vkauehu4qCJna6Q4YRn'
```
Usando Django, você pode se autenticar na sua views em questão, quando for usar a pagarme, ou no settings.py, ja que é o arquivo que é carregado sempre antes de qualquer coisa (anyway, mais pra frente tentaria por exemplos).

A partir do momento que você se autenticou já pode usar as funções e métodos da biblioteca.

Aconselho dar uma lida na documentação oficial, porque ela é realmente boa e simples.

### Exemplos

##### Criando um cartão - https://docs.pagar.me/reference#criando-um-cartao
```python
import pagarme

pagarme.authentication_key('SUA_API_KEY')
# os dados são sempre em strings e sempre em dicionario com
# chave - valor

# vale acessar a api e ver exatamente os campos que a api espera
# 	como parâmetro

card_data = {
    # data de exp é '1122' e não '11/22' ou '11-22'
    "card_expiration_date": "1122",
    # o numero do cartão nao tem espaço nem nada, só os numeros
    "card_number": "4018720572598048",
    "card_cvv": "123",
    "card_holder_name": "Cersei Lannister"
}
card_created = pagarme.card.create(card_data)
print (card_created)
# é um api realmente bem amigável
```

##### Criando uma transação - https://docs.pagar.me/reference#criar-transacao

Uma transação tem basicamente:
	-- Cartão e seus dados,
	-- Comprador e seus dados,
	-- Vendedor e seus dados,
	-- Itens da compra

Vale acessar a API para entender melhor quais campos são obrigatórios e tals
```python
import pagarme

pagarme.authentication_key('SUA_API_KEY')

params = {
    # ps, o amount é em centavos, sempre
    "amount": "21000",
    "card_number": "4111111111111111",
    "card_cvv": "123",
    "card_expiration_date": "0922",
    "card_holder_name": "Morpheus Fishburne",
    "customer": {
      "external_id": "#3311",
      "name": "Morpheus Fishburne",
      "type": "individual",
      "country": "br",
      "email": "mopheus@nabucodonozor.com",
      "documents": [
        {
          "type": "cpf",
          "number": "30621143049"
        }
      ],
      "phone_numbers": ["+5511999998888", "+5511888889999"],
      # repare como a data é bem especifica: AAAA-MM-DD, não pode ser de outro jeito
      "birthday": "1965-01-01"
    },
    "billing": {
      "name": "Trinity Moss",
      "address": {
        "country": "br",
        "state": "sp",
        "city": "Cotia",
        "neighborhood": "Rio Cotia",
        "street": "Rua Matrix",
        "street_number": "9999",
        "zipcode": "06714360"
      }
    },
    "shipping": {
      "name": "Neo Reeves",
      "fee": "1000",
      "delivery_date": "2000-12-21",
      "expedited": True,
      "address": {
        "country": "br",
        "state": "sp",
        "city": "Cotia",
        "neighborhood": "Rio Cotia",
        "street": "Rua Matrix",
        "street_number": "9999",
        "zipcode": "06714360"
      }
    },
    "items": [
      {
        "id": "r123",
        "title": "Red pill",
        "unit_price": "10000",
        "quantity": "1",
        "tangible": True
      },
      {
        "id": "b123",
        "title": "Blue pill",
        "unit_price": "10000",
        "quantity": "1",
        "tangible": True
      }
    ]
}

trx = pagarme.transaction.create(params)

print(trx)
```

## Exemplos no Leve Carne

## Transação

Pegando os dados (tudo isso ta dentro de uma views, processando o post request)

```python
card_number = request.POST.get('card-number')
card_name = request.POST.get('first_name') + request.POST.get('last_name')
card_month = request.POST.get('card-month')
card_year = request.POST.get('card-year')
card_cvv = request.POST.get('card-cvv')
card_exp = str(card_month) + str(card_year)

user = request.user
usuario = user.usuario
nome = usuario.nome
sobrenome = usuario.sobrenome
data_nascimento = usuario.data_nascimento
email = user.email
endereco = usuario.endereco_set.all().get(selecionado=True)
CEP = usuario.CEP
CPF = usuario.cpf
comentarios = request.POST['comentarios']
carrinho = usuario.carrinho
produtos = carrinho.produtos.all()
valor = carrinho.valor_total
valor_centavos = valor * 100
```

Criando dicionario

```python
trx_data = {
    'capture' : 'false',
    # lembrando, o amount precisa ser em centavos e EXATO, não pode ser um numero quebrado
    'amount': str(int(valor_centavos)),
    'card_number': card_number,
    'card_cvv': card_cvv,
    'card_expiration_date': card_exp,
    'card_holder_name': card_name,
    'customer': {
      'external_id': str(usuario.pk),
      'name': nome,
      'type': 'individual',
      'country': 'br',
      'email' : email,
      'documents': [
        {
          'type': 'cpf',
          'number' : str(CPF)
        },
      ],
      'phone_numbers': [usuario_telefone],
      'birthday': usuario_data
    },
    'billing': {
        'name': carrinho.lojista.nome_responsavel,
        'address': {
            'country': 'br',
            'state': carrinho.lojista.uf,
            'city': carrinho.lojista.cidade,
            'neighborhood': carrinho.lojista.bairro,
            'street': carrinho.lojista.rua,
            'street_number': str(carrinho.lojista.numero),
            'zipcode': lojista_zipcode
        }
    },
    'items': [
    ]
}
```

## Conta bancária

```python
class CadastroContaBancaria(View):
    template_name = 'cadastro_banco.html'
    def get(self, request):
	# ... 
        return render(request, self.template_name, context)

    def post(self, request):
        context = {}
        post_data = request.POST
        bank_account_data = {
            'agencia': post_data.get('agencia'),
            'agencia_dv': post_data.get('agencia_dv'),
            'bank_code': post_data.get('bank_code'),
            'conta': post_data.get('conta'),
            'conta_dv': post_data.get('conta_dv'),
            'document_number': post_data.get('document_number').replace('.', '').replace('/', ''),
            'legal_name': post_data.get('legal_name')
        }

        recipient_data = {
            'transfer_day': '1',
            'transfer_enabled': 'true',
            'transfer_interval': 'monthly',
            'bank_account': bank_account_data,
        }

        bank_account = pagarme.bank_account.create(bank_account_data)
        recipient = pagarme.recipient.create(recipient_data)
        # ...
        # ...
```

## Vale sempre olhar as respostas da API no site, ela retorna um JSON com status, valores, ids, etc

### Algumas outras explicações

Na pagarme, temos basicamente 3 agentes:
-- Consumidor representado pela *Classe Customer*
-- Prestador de serviço representado pela *Classe Recipient* (só para -- Marketplaces, onde tem um terceiro a gente)
-- O administrador do sistema (geralmente nosso cliente) e ele é o * default recipient* do sistema

#### O fluxo de pagamento funciona da seguinte maneira:

Consumidor faz uma transação para a pagarme;
A pagarme processa o pagamento, retira devidas taxas, etc;
E então ela faz transferências para os prestadores de serviço, caso eles existam;

Esse cenário é de um marketplace, como o Leve Carne por exemplo,
mas aí você pode se perguntar, "tá, mas dado um serviço de 100 reais, quanto fica para o cliente em si (que ta fazendo projeto com a pj) e o prestador de serviço?"
No Leve Carne seria: "Quanto o Marino (nosso cliente) vai ganhar para cada Picanha vendida pelo Senhor João do Açougue X?"
# PS: LEMBRANDO QUE ISSO É PARA UM MAKETPLACE, BJS
Na hora de criar a transação, você pode passar um split_rules, dessa maneira:

```python
            'split_rules': [
                {
                    'recipient_id': default_recipient['test'], # id da conta do Marino na pagarme
                    'percentage': 10, # porcentagem tem que ser um inteiro (por exemplo 10)
                    'liable': 'true',
                    'charge_processing_fee': 'true'
                },
                {
                    'recipient_id': lojista_em_questao.recebedor_pagarme_id, # id do lojista em questao (pegar do cadastro bancario dele)
                    'percentage': 90, # porcentagem tem que ser um inteiro (por exemplo 90)
                    'liable': 'true',
                    'charge_processing_fee': 'false'
                }
            ]
```

Agora se o usuário vai pagar pro cliente em si, e não tem um terceiro nessa brisa, é só ignorar isso e boa, fazer a transação da maneira mais simples e é isso.

Taxas da pagarme são simples de achar no site e podem ser negociadas.
# Deploy

Repare que na Dashboard da pagarme sempre estamos em ambiente de testes, transações de teste e tals, isso porque a API_KEY deles é de teste,
tem até '_test' no nome.

Para o deploy definitivo, para produção, o que precisa mudar é apenas a API_KEY, para conseguir essa key definitiva de '_production' é preciso completar o cadastro na Dashboard da pagarme e aguardar a revisão.

# Página para checkout
 
A pagarme tem uma pagina de checkout pronta aqui: https://docs.pagar.me/docs/inserindo-o-checkout

É por exemplo aí que voce pode usar o outra key da pagarme, digamos que a principal você nao pode expor NUNCA, mas aí pode ir no front as vezes, como no exemplo do site. 

Ela cria um modal que se conecta com o site deles e faz todo o processo de transação, retornando um ID, que é o id da transação, é dahora, tem toda uma animação de sucesso e erro.

##### PS IMPORTANTE, CASO VOCÊ PRECISE FAZER UMA AUTORIZAÇÃO PRÉVIA, PARA DEPOIS FAZER A CAPTURA DA TRANSAÇÃO, VOCÊ NÃO PODE USAR ESSE CHECKOUT, VOCE TEM QUE FAZER A TRANSAÇÃO TODINHA NO BACKEND

é isso galerinha, leiam a doc deles, é bem boa, e tentem ir se baseando na documentação deles, porque pode ser que alguma atualização mude alguma coisa e tals, mas é isso, espero ter dado um panorama melhor sobre essa plataforma de pagamento, bjs no coração de vocês e bora codar :3




