# Mecanismo de validação de dados

Este validador foi inspirado no Validator utilizado pelo Laravel, mas sua implementação foi adaptada para atender as necessidades dos processos
do sistema Receiv.

## Sumário

- [Validador de dados da Receiv](#validador-de-dados-da-receiv)
  * [Utilização](#utilização)
  * [Sintaxe das regras](#sintaxe-das-regras)
  * [Customização de mensagens](#customização-de-mensagens)
  * [Regras de validação](#regras-de-validação)
  * [Documentação](#documentação)
    + [validate(array $dados, array $regras)](#validatearray-dados-array-regras)
    + [fails()](#fails)
    + [setMsg(string $campo, array $regras)](#setmsgstring-campo-array-regras)
    + [getMessages()](#getmessages)

--------
## Utilização

```php
require_once "vendor/autoload.php";

use BWCS\Validator\Validator;

// Instanciação do validador
$validator = new Validator();

// Executando a validação
$validator->validate($_POST, [
    'credor_id'     => 'required|integer|exists:tb_credor,credor_id|stop',
    'email'         => 'required|email|unique:tb_usuario,e_mail|stop',
    'password'      => 'required|confirmed|min:8',
    'data_inicial'  => 'required|date_between:-30 days,-1 days'
]);

// Obtendo o resultado da validação e printando na tela
if($validator->fails()) {
    print_r($validator->getMessages());
}
```
## Sintaxe das regras

Um campo pode ter mais de uma regra, sendo separadas por um _| (pipe)_.

**Ex.:**
```php
required|string|stop
```
Quando uma regra receber parametros, deverão vir após um _:_ e os parâmetros deverão ser separados por vígula.
**Ex.:**
```php
required|integer|exists:tb_credor,credor_id
```

## Customização de mensagens
A customização de mensagem de erro de validação pode ser feita no método [setMsg](#setmsgstring-campo-array-regras) podendo ser definidas mensagens específicas para cada tipo de erro.
Caso haja mais de uma regra mas seja definida uma mensage customizada para apenas uma, as outras regras retornarão a mensagem de erro padrão.
**Obs.:** O método [setMsg](#setmsgstring-campo-array-regras) deve ser chamado antes do método [validate](#validatearray-dados-array-regras). 
Ex.:
```php
$validator->setMsg('email', [
    'required' => 'O e-mail deve ser informado',
    'email'    => 'E-mail inválido',
    'unique'   => 'Este e-mail não está disponível para registro'
]);

$validator->setMsg('nascimento', [
    'date_between' => 'O dependente deve ter entre 12 e 18 anos de idade'
]);

$validator->validate($_POST, [
    'email'      => 'required|email|unique:tb_usuario,e_mail|stop',
    'nascimento' => 'required|date_between:-18 years,-12 years'
]);
```

## Regras de validação

| Regra  | Descrição |
| ------ | --------- |
| **required**                | O campo deve existir e ter algum valor não nulo. |
| **stop**                    | Interrompe a execução das regras seguintes caso um campo não seja válido |
| **integer**                 | O campo deve ter um valor inteiro. |
| **numeric**                 | O campo deve ter um valor númerico. |
| **bool**                    | O campo deve um valor booleano. Os valores aceitos são true, false, 1, 0, "1", e "0". |
| **checked**                 | O campo deve ter um dos valores "yes", "on", true, "true", 1, "1". |
| **not_checked**             | O campo pode ter um dos valores "yes", "on", true, "true", 1, "1". |
| **alpha**                   | Deve conter apenas caracteres alfabéticos |
| **alpha_numeric**           | Deve conter apenas caracteres alfabéticos e numéricos |
| **alpha_dash**              | Deve conter apenas caracteres alfabéticos, **- (hífen)** ou  **_ (underscore)** |
| **email**                   | O campo deve ser um email válido |
| **confirmed**               | O campo deve ter um campo de confirmação. O campo de confirmação deve ser o nome do campo principal seguido de **_confirmation** |
| **contains:_string_**       | Valida se o valor do campo contém o uma _string_ |
| **not_contains:_string_**   | Valida se o valor do campo **não** contém o uma _string_ |
| **json**                    | Valida se o valor fornecido é um json válido |
| **url**                     | Valida se o valor fornecido é uma url válida |
| **min:_valor_**             | Se o valor do campo for numérico verifica se um valor do campo é maior ou igual o valor definido na regra. Se o valor do campo for uma string verifica se o compromento é maior ou igual o valor definido na regra |
| **max:_valor_**             | Se o valor do campo for numérico verifica se um valor do campo é menor ou igual o valor definido na regra. Se o valor do campo for uma string verifica se o compromento é menor ou igual o valor definido na regra |
| **between:_min,max_**       | Confere se o valor é maior ou igual a _min_ e menor e igual a _max_ |
| **date**                    | Valida se o valor do campo é uma data válida no formato AAAA-MM-DD |
| **date_format:_formato_**   | Valida se o valor do campo é uma data válida no formato definindo. Internamente utiliza a função [strtotime](https://www.php.net/manual/en/function.strtotime) do php então deve seguir os [formatos suportados](https://www.php.net/manual/en/datetime.formats.php) |
| **before:_data_**           | Confere se o valor do campo é uma data anterior à _data_ definida na regra |
| **after:_data_**            | Confere se o valor do campo é uma data posterior à _data_ definida na regra |
| **date_between:_min,max_**  | Valida se o valor do campo é uma data válida e em um determinado período |
| **in:_Sim,Não,Talvez_**     | Valida se o valor do campo está em uma lista de valores informados na regra |
| **not_in:_Sim,Não,Talvez_** | Valida se o valor do campo **não** está em uma lista de valores informados na regra |
| **exists:_table,column_**   | Valida se o valor existe em uma coluna de uma tabela informados na regra. |
| **unique:_table,column_**   | O valor deve ser único na tabela/coluna informadas. Retorna inválido caso o valor do campo já exista.  |


## Métodos
### validate(array $dados, array $regras)
Executa a validação baseada em uma lista de regras passada para cada campo.

- **dados**(_array_): Lista de dados a serem validados com cada item no formato 'campo' => 'dados';
- **regras**(_array_): Lista de regras de validação seguindo o formato 'campo' => 'regras';
### fails()
Retorna _true_ se a validação ocorreu com algum erro.

### setMsg(string $campo, array $regras)
Define mensagens customizadas para cada tipo de erro em um campo.

- **campo**(_array_): Campo ao qual a lista de mensagens será aplicada.
- **regras**(_array_): Lista de de mensagens com cada item seguindo o formato 'regra' => 'Mensagem';


### getMessages()