**NÃO LEIA ESTE ARQUIVO NO GITHUB, OS GUIAS SÃO PUBLICADOS NO https://guiarails.com.br.**
**DO NOT READ THIS FILE ON GITHUB, GUIDES ARE PUBLISHED ON https://guides.rubyonrails.org.**

Atualizando o Ruby on Rails
=======================

Este guia fornece os passos a serem seguidos quando você for atualizar suas aplicações
para uma versão mais nova do Ruby on Rails. Estes passos também estão disponíveis em guias de *releases* individuais.

--------------------------------------------------------------------------------

Conselho Geral
--------------

Antes de tentar atualizar uma aplicação existente, você deve ter certeza que possui uma boa razão para fazê-lo. Então tenha em mente alguns fatores: a necessidade de novas funcionalidades, a crescente dificuldade de encontrar suporte para código mais antigo, seu tempo disponível e habilidades, entre outros.

### Cobertura de Testes

A melhor maneira de garantir que sua aplicação ainda funciona após a atualização é possuir uma boa cobertura de testes antes de começar o processo. Se você não tiver testes automatizados para a maior parte de sua aplicação, será necessário gastar algum tempo realizando testes manuais de todas as partes alteradas. No caso da atualização do Rails, isso significa cada umas das funcionalidades dentro da aplicação. Faça a si mesmo um favor e tenha certeza de ter uma boa cobertura de teste **antes** de iniciar uma atualização.

### O Processo de Atualização

Quando estiver atualizando a versão do Rails, o melhor é ir devagar, uma versão *Minor* por vez, para fazer bom uso dos avisos de depreciação. As versões do Rails são numeradas da maneira *Major*.*Minor*.*Patch*. Versões *Major* e *Minor* têm permissão para alterar API pública, isso pode causar erros em sua aplicação. Versões *Patch* incluem apenas correções de *bug*, e não alteram nenhuma API pública.

O processo deve correr da seguinte maneira:

1. Escreva os testes e garanta que eles passem.
2. Atualize para a última versão *Patch* após a versão atual de seu projeto.
3. Conserte os testes e funcionalidades depreciadas.
4. Atualize para a última versão *Patch* da versão *Minor* seguinte.

Repita este processo até chegar na versão desejada do Rails. Cada vez que você mudar entre versões, você precisará alterar a versão do Rails no `Gemfile` (possivelmente também as versões de outras *gems*) e executar um `bundle update`. Após, execute a tarefa de atualização mencionada abaixo para atualizar arquivos de configuração, então execute os testes.

Você pode encontrar a lista com todas as versões liberadas do Rails [aqui](https://rubygems.org/gems/rails/versions)

### Versões do Ruby

Rails geralmente se mantém próximo à versão mais recente do Ruby quando é liberado:

* Rails 6 requer Ruby 2.5.0 ou posterior.
* Rails 5 requer Ruby 2.2.2 ou posterior.
* Rails 4 prefere Ruby 2.0 e requer 1.9.3 ou posterior.
* Rails 3.2.x é a última *branch* a suportar Ruby 1.8.7.
* Rails 3 e posteriores requerem Ruby 1.8.7 ou posteriores. O suporte para todas as versões anteriores do Ruby foi descontinuado. Você deve atualizar o quanto antes.

Dica: O Ruby 1.8.7 p248 e p249 possuem *bugs* que travam o Rails. Ruby *Enterprise Edition* possui correções para esses erros desde a versão 1.8.7-2010.02. Na Versão 1.9, Ruby 1.9.1 não é utilizável pois ela falha logo de início devido a problemas de memória, então se você quiser utilizar a versão 1.9.x, pule diretamente para 1.9.3 para evitar problemas.

### A Tarefa de Atualização

Rails fornece o comando `app:update` (`rake rails:update` na versão 4.2 e anteriores). Execute este comando após atualizar a versão do Rails no `Gemfile`. Isto lhe ajudará na criação de novos arquivos e na alteração de arquivos antigos em uma sessão interativa.

```bash
$ bin/rails app:update
   identical  config/boot.rb
       exist  config
    conflict  config/routes.rb
Overwrite /myapp/config/routes.rb? (enter "h" for help) [Ynaqdh]
       force  config/routes.rb
    conflict  config/application.rb
Overwrite /myapp/config/application.rb? (enter "h" for help) [Ynaqdh]
       force  config/application.rb
    conflict  config/environment.rb
...
```

Não esqueça de revisar a diferença, para verificar se houveram mudanças inesperadas.

### Configurar Padrões de Framework

A nova versão do Rails pode ter configurações padrão diferentes da versão anterior. No entanto, após seguir os passos descritos acima, sua aplicação ainda estaria rodando com configurações padrão da versão **anterior** do Rails. Isso porque o valor para `config.load_defaults` em `config/application.rb` ainda não foi alterado.

Para permitir que você atualize para novos padrões um a um, a tarefa de atualização cria um arquivo `config/initializers/new_framework_defaults.rb`. Uma vez que a aplicação esteja pronta para rodar com as novas configurações padrão, você pode remover este arquivo e trocar o valor de `config.load_defaults`.

Atualizando do Rails 6.0 para o Rails 6.1
-------------------------------------

Para mais informações sobre as mudanças feitas no Rails 6.1 consulte as [notas de lançamento](6_1_release_notes.html).

### `Rails.application.config_for` o valor de retorno não oferece mais suporte para acesso com chaves *String*.

Dado um arquivo de configuração como este:

```yaml
# config/example.yml
development:
  options:
    key: value
```

```ruby
Rails.application.config_for(:example).options
```

Isso costumava retornar um *hash* no qual você podia acessar valores com chaves *String*. Isso foi descontinuado no 6.0 e agora não funciona mais.

Você pode chamar `with_indifferent_access` no valor de retorno de` config_for` se ainda quiser acessar valores com chaves *String*, por exemplo:

```ruby
Rails.application.config_for(:example).with_indifferent_access.dig('options', 'key')
```

### Respostas do tipo de conteúdo ao utilizar `respond_to#any`

O cabeçalho (*header*) do tipo de conteúdo (*Content-Type*) retornado na resposta pode ser diferente do que o Rails 6.0 retornou,
mais especificamente se sua aplicação usa o formato `respond_to { |format| format.any }`.
O tipo de conteúdo será baseado no bloco fornecido e não no formato da solicitação.

Exemplo:

```ruby
def my_action
  respond_to do |format|
    format.any { render(json: { foo: 'bar' }) }
  end
end
```

```ruby
get('my_action.csv')
```

O comportamento anterior era retornar um tipo de conteúdo de resposta `text/csv` que é impreciso uma vez que uma resposta JSON está sendo renderizada.
O comportamento atual retorna corretamente o tipo de conteúdo de uma resposta `application/json`.

Se sua aplicação depende do comportamento incorreto anterior, você é incentivado a especificar
quais formatos sua ação aceita, ou seja.

```ruby
format.any(:xml, :json) { render request.format.to_sym => @people }
```

### `ActiveSupport::Callbacks#halted_callback_hook` agora recebe um segundo argumento

*Active Support* permite que você substitua o `halted_callback_hook` sempre que um retorno de chamada
pare a sequência. Este método agora recebe um segundo argumento que é o nome do retorno de chamada que está sendo interrompido.
Se você tiver classes que substituem esse método, certifique-se de que ele aceite dois argumentos. Observe que isso é uma mudança
significativa sem um ciclo de depreciação anterior (por motivos de desempenho).

Exemplo:

```ruby
class Book < ApplicationRecord
  before_save { throw(:abort) }
  before_create { throw(:abort) }

  def halted_callback_hook(filter, callback_name) # => Este método agora aceita 2 argumentos em vez de 1
    Rails.logger.info("Book couldn't be #{callback_name}d")
  end
end
```

### O método de classe `helper` nos *controllers* usa `String#constantize`

Conceitualmente antes do Rails 6.1

```ruby
helper "foo/bar"
```

resultou em

```ruby
require_dependency "foo/bar_helper"
module_name = "foo/bar_helper".camelize
module_name.constantize
```

Agora ele faz isso:

```ruby
prefix = "foo/bar".camelize
"#{prefix}Helper".constantize
```

Essa mudança é compatível com as versões anteriores para a maioria das aplicações, nesse caso, você não precisa fazer nada.

Tecnicamente, no entanto, os controllers podem configurar `helpers_path` para apontar para um diretório em `$LOAD_PATH` que não estava nos caminhos de carregamento automático. Esse caso de uso não é mais compatível com o uso imediato. Se o módulo auxiliar não for auto-carregável, a aplicação é responsável por carregá-lo antes de chamar o `helper`.

### Redirecionamento para HTTPS vindo de HTTP agora usará o código de status 308 HTTP

O código de status HTTP padrão usado em `ActionDispatch::SSL` ao redirecionar solicitações não GET/HEAD de HTTP para HTTPS foi alterado para `308` conforme definido em https://tools.ietf.org/html/rfc7538.

### Active Storage agora requer Processamento de Imagem

Ao processar variantes no Active Storage, agora é necessário ter a *gem* [image_processing](https://github.com/janko-m/image_processing) empacotada em vez de usar diretamente `mini_magick`. O processamento de imagem é configurado por padrão para usar `mini_magick` nos bastidores, então a maneira mais fácil de atualizar é substituindo a gem `mini_magick` pela gem `image_processing` e certificando-se de remover o uso explícito de `combine_options`, uma vez que não é mais necessário.

Para facilitar a leitura, você pode desejar alterar as chamadas `resize` brutas para macros `image_processing`. Por exemplo, em vez de:

```ruby
video.preview(resize: "100x100")
video.preview(resize: "100x100>")
video.preview(resize: "100x100^")
```

você pode fazer respectivamente:

```ruby
video.preview(resize_to_fit: [100, 100])
video.preview(resize_to_limit: [100, 100])
video.preview(resize_to_fill: [100, 100])
```

Atualizando do Rails 5.2 para o Rails 6.0
-------------------------------------

Para mais informações sobre as mudanças feitas no Rails 6.0 consulte as [notas de lançamento](6_0_release_notes.html).

### Usando Webpacker

[Webpacker](https://github.com/rails/webpacker)
é o compilador *JavaScript* padrão para Rails 6. Mas se você estiver atualizando a aplicação, ele não é ativado por padrão.
Se você quiser usar o *Webpacker*, adicione ele em seu *Gemfile* e instale:

```ruby
gem "webpacker"
```

```bash
$ bin/rails webpacker:install
```

### Forçar SSL

O método `force_ssl` nos *controllers* foi descontinuado e será removido no
Rails 6.1. Você é encorajado a habilitar `config.force_ssl` para impor conexões
HTTPS ao longo de sua aplicação. Se você precisar isentar certos *endpoints*
do redirecionamento, você pode usar `config.ssl_options` para configurar esse comportamento.

### Propósito (*Purpose*) e metadados de expiração agora estão incorporados em cookies assinados e criptografados para maior segurança

Para melhorar a segurança, o Rails incorpora os metadados de propósito e expiração dentro do valor de cookies criptografados ou assinados.

Rails pode então impedir ataques que tentam copiar o valor assinado/criptografado
de um *cookie* e usá-lo como o valor de outro *cookie*.

Esses novos metadados incorporados tornam esses *cookies* incompatíveis com versões do Rails anteriores a 6.0.

Se você deseja que seus *cookies* sejam lidos pelo Rails 5.2 e anteriores, ou ainda está validando seu *deploy* do 6.0 e deseja ser capaz de reverter (*rollback*)
`Rails.application.config.action_dispatch.use_cookies_with_metadata` para `false`.

### Todos os pacotes npm foram movidos para o escopo `@rails`

Se você estava anteriormente carregando qualquer um dos pacotes `actioncable`, `activestorage`,
ou `rails-ujs` através de npm/yarn, você deve atualizar os nomes destas
dependências antes de atualizá-los para o `6.0.0`:

```
actioncable   → @rails/actioncable
activestorage → @rails/activestorage
rails-ujs     → @rails/ujs
```

### Mudanças na API do *Action Cable JavaScript*

O pacote *Action Cable JavaScript* foi convertido do *CoffeeScript*
para *ES2015*, e agora publicamos o código-fonte via distribuição pelo npm.

Esta versão inclui algumas mudanças importantes para partes opcionais da
*API JavaScript Action Cable*:

- A configuração do adaptador *WebSocket* e do adaptador *logger* foi movida
  das propriedades de `ActionCable` para as propriedades de `ActionCable.adapters`.
  Se você estiver configurando esses adaptadores, você precisará fazer
  estas alterações:

    ```diff
    -    ActionCable.WebSocket = MyWebSocket
    +    ActionCable.adapters.WebSocket = MyWebSocket
    ```

    ```diff
    -    ActionCable.logger = myLogger
    +    ActionCable.adapters.logger = myLogger
    ```

- Os métodos `ActionCable.startDebugging()` e `ActionCable.stopDebugging()`
  foram movidos e substituídos pela propriedade
  `ActionCable.logger.enabled`. Se você estiver usando esse métodos, você
  precisará fazer estas alterações:

    ```diff
    -    ActionCable.startDebugging()
    +    ActionCable.logger.enabled = true
    ```

    ```diff
    -    ActionCable.stopDebugging()
    +    ActionCable.logger.enabled = false
    ```

### `ActionDispatch::Response#content_type` agora retorna o cabeçalho (*header*) do tipo de conteúdo (*Content-Type*) sem modificação

Anteriormente, o valor de retorno de `ActionDispatch::Response#content_type` NÃO continha a parte do conjunto de caracteres.
Este comportamento foi alterado para incluir também a parte do conjunto de caracteres omitida anteriormente.

Se você quiser apenas o tipo *MIME*, use `ActionDispatch::Response#media_type` em seu lugar.

Antes:

```ruby
resp = ActionDispatch::Response.new(200, "Content-Type" => "text/csv; header=present; charset=utf-16")
resp.content_type #=> "text/csv; header=present"
```

Depois:

```ruby
resp = ActionDispatch::Response.new(200, "Content-Type" => "text/csv; header=present; charset=utf-16")
resp.content_type #=> "text/csv; header=present; charset=utf-16"
resp.media_type   #=> "text/csv"
```

### Carregamento Automático

A configuração padrão para Rails 6

```ruby
# config/application.rb

config.load_defaults 6.0
```

ativa o modo de carregamento automático `zeitwerk` no CRuby. Nesse modo, o carregamento automático, o recarregamento e o carregamento antecipado são gerenciados pelo [Zeitwerk](https://github.com/fxn/zeitwerk).

#### API Pública

Em geral, as aplicações não precisam usar a API do *Zeitwerk* diretamente. Rails configura as coisas de acordo com o contrato existente: `config.autoload_paths`,`config.cache_classes`, etc.

Embora as aplicações devam seguir essa interface, o objeto do carregador *Zeitwerk* atual pode ser acessado como

```ruby
Rails.autoloaders.main
```

Isso pode ser útil se você precisar pré-carregar STIs ou configurar um *inflector* customizado, por exemplo.

#### Estrutura do Projeto

Se a aplicação que está sendo atualizada for carregada automaticamente de forma correta, a estrutura do projeto já deve ser compatível.

No entanto, o modo `clássico` entende nomes de arquivos com (`underscore`), enquanto o modo `zeitwerk` entende nomes de arquivos (`camelize`). Esses *helpers* nem sempre são inversos entre si, especialmente se houver acrônimos envolvidos. Por exemplo, `"FOO".underscore` é `"foo"`, mas `"foo".camelize` é `"Foo"`, não `"FOO "`.

A compatibilidade pode ser verificada com a tarefa `zeitwerk:check`:

```bash
$ bin/rails zeitwerk:check
Hold on, I am eager loading the application.
All is good!
```

#### *require_dependency*

Todos os casos de uso conhecidos de `require_dependency` foram eliminados, você deve executar o *grep* no projeto e excluí-los.

Se sua aplicação tiver STIs, verifique a seção no guia [Carregamento Automático e Recarregando Constantes (Modo Zeitwerk)](autoloading_and_reloading_constants.html#single-table-inheritance).

#### Nomes qualificados nas definições de classe e módulo

Agora você pode usar *constant paths* de forma robusta nas definições de classe e módulo:

```ruby
# O carregamento automático no corpo desta classe corresponde à semântica Ruby agora.
class Admin::UsersController < ApplicationController
  # ...
end
```

Um problema a ter em conta é que, dependendo da ordem de execução, o auto carregamento clássico pode às vezes ser capaz de carregar automaticamente `Foo::Wadus` em

```ruby
class Foo::Bar
  Wadus
end
```

Isso não corresponde à semântica Ruby porque `Foo` não está no aninhamento e não funcionará no modo `zeitwerk`. Se você encontrar esse caso, você pode usar o nome qualificado `Foo::Wadus`:

```ruby
class Foo::Bar
  Foo::Wadus
end
```

ou adicione `Foo` ao aninhamento:

```ruby
module Foo
  class Bar
    Wadus
  end
end
```

#### (*Concerns*)

Você pode carregar automaticamente e antecipadamente a partir de uma estrutura padrão como

```
app/models
app/models/concerns
```

Nesse caso, `app/models/concerns` é considerado um diretório raiz (porque pertence aos caminhos de carregamento automático) e é ignorado como *namespace*. Portanto, `app/models/concern/foo.rb` deve definir `Foo`, não `Concerns::Foo`.

O *namespace* `Concerns::` funcionou com o carregamento automático clássico como um efeito colateral da implementação, mas não foi realmente um comportamento pretendido. Uma aplicação que usa `Concerns::` precisa renomear essas classes e módulos para poder rodar no modo `zeitwerk`.

#### Tendo `app` nos caminhos de carregamento automático

Alguns projetos querem algo como `app/api/base.rb` para definir `API::Base`, e adicionar `app` aos caminhos de carregamento automático para fazer isso no modo `clássico`. Já que Rails adiciona todos os subdiretórios de `app` aos caminhos de carregamento automático automaticamente, temos outra situação em que há diretórios raiz aninhados, de forma que a configuração não funciona mais. Princípio semelhante que explicamos acima com `concerns`.

Se quiser manter essa estrutura, você precisará excluir o subdiretório dos caminhos de carregamento automático em um inicializador:

```ruby
ActiveSupport::Dependencies.autoload_paths.delete("#{Rails.root}/app/api")
```

#### Constantes carregadas automaticamente e *namespaces* explícitos

Se um *namespace* for definido em um arquivo, como `Hotel` está aqui:

```
app/models/hotel.rb         # Defines Hotel.
app/models/hotel/pricing.rb # Defines Hotel::Pricing.
```

a constante `Hotel` deve ser definida usando as palavras-chave `class` ou `module`. Por exemplo:

```ruby
class Hotel
end
```

é bom.

Alternativas como

```ruby
Hotel = Class.new
```

ou

```ruby
Hotel = Struct.new
```

não funcionará, objetos filhos como `Hotel::Pricing` não serão encontrados.

Essa restrição se aplica apenas a *namespaces* explícitos. Classes e módulos que não definem um *namespace* podem ser definidos usando esses idiomas.

#### Um arquivo, uma constante (no mesmo nível superior)

No modo `classic`, você pode definir tecnicamente várias constantes no mesmo nível superior e ter todas elas recarregadas. Por exemplo, dado

```ruby
# app/models/foo.rb

class Foo
end

class Bar
end
```

enquanto `Bar` não pôde ser carregado automaticamente, o carregamento automático de `Foo` marcaria `Bar` como carregado automaticamente também. Este não é o caso no modo `zeitwerk`, você precisa mover `Bar` para seu próprio arquivo `bar.rb`. Um arquivo, uma constante.

Isso afeta apenas as constantes no mesmo nível superior do exemplo acima. Classes e módulos internos são adequados. Por exemplo, considere

```ruby
# app/models/foo.rb

class Foo
  class InnerClass
  end
end
```

Se a aplicação recarregar `Foo`, ela irá recarregar `Foo::InnerClass` também.

#### *Spring* e o ambiente `test`

*Spring* recarrega o código da aplicação se algo mudar. No ambiente `test`, você precisa habilitar o recarregamento para que funcione:

```ruby
# config/environments/test.rb

config.cache_classes = false
```

Caso contrário, você obterá este erro:

```
reloading is disabled because config.cache_classes is true
```

#### *Bootsnap*

O *Bootsnap* deve ter pelo menos a versão 1.4.2.

Além disso, o *Bootsnap* precisa desabilitar o cache *iseq* devido a um bug no interpretador se estiver executando o Ruby 2.5. Certifique-se de depender de pelo menos Bootsnap 1.4.4 nesse caso.

#### `config.add_autoload_paths_to_load_path`

O novo ponto de configuração

```ruby
config.add_autoload_paths_to_load_path
```

é `true` por padrão para compatibilidade com versões anteriores, mas permite que você opte por não adicionar os caminhos de carregamento automático a `$LOAD_PATH`.

Isso faz sentido na maioria das aplicações, já que você nunca deve requerer um arquivo em `app/models`, por exemplo, e o *Zeitwerk* só usa nomes de arquivo absolutos internamente.

Ao optar pela exclusão, você otimiza as pesquisas ao `$LOAD_PATH` (menos diretórios para verificar) e economiza o trabalho do *Bootsnap* e o consumo de memória, já que não é necessário construir um índice para esses diretórios.

#### *Thread-safety*

No modo clássico, o carregamento automático constante não é *thread-safe*, embora o Rails tenha travas, por exemplo, para tornar as solicitações da web *thread-safe* quando o carregamento automático está habilitado, como é comum no ambiente de desenvolvimento.

O carregamento automático constante é *thread-safe* no modo `zeitwerk`. Por exemplo, agora você pode carregar automaticamente em scripts *multi-threaded* executados pelo comando `runner`.

#### *Globs* em *config.autoload_paths*

Cuidado com configurações como

```ruby
config.autoload_paths += Dir["#{config.root}/lib/**/"]
```

Cada elemento de `config.autoload_paths` deve representar o *namespace* de nível superior (`Object`) e eles não podem ser aninhados em consequência (com exceção dos diretórios `concerns` explicados acima).

Para corrigir isso, basta remover os curingas (*wildcards*):

```ruby
config.autoload_paths << "#{config.root}/lib"
```

#### Carregamento rápido (*Eager loading*) e carregamento automático são consistentes

No modo `clássico`, se `app/models/foo.rb` define `Bar`, você não será capaz de carregar automaticamente aquele arquivo, mas o carregamento rápido funcionará porque carrega os arquivos recursivamente às cegas. Isso pode ser uma fonte de erros se você testar as coisas primeiro com carregamento rápido; a execução pode falhar no carregamento automático posterior.

No modo `zeitwerk` ambos os modos de carregamento são consistentes, eles falham e erram nos mesmos arquivos.

#### Como usar o Carregamento Automático Clássico no Rails 6

As aplicações podem carregar os padrões do Rails 6 e ainda usar o carregamento automático clássico definindo `config.autoloader` desta forma:

```ruby
# config/application.rb

config.load_defaults 6.0
config.autoloader = :classic
```

Ao usar o Carregamento Automático Clássico na aplicação Rails 6, é recomendado definir o nível de simultaneidade (*concurrency*) como 1 no ambiente de desenvolvimento, para os servidores web e processadores de segundo plano, devido às questões de *thread-safety*.

### Alteração de comportamento de atribuição do *Active Storage*

Com os padrões de configuração para Rails 5.2, atribuir a uma coleção de anexos declarados com `has_many_attached` acrescenta novos arquivos:

```ruby
class User < ApplicationRecord
  has_many_attached :highlights
end

user.highlights.attach(filename: "funky.jpg", ...)
user.highlights.count # => 1

blob = ActiveStorage::Blob.create_after_upload!(filename: "town.jpg", ...)
user.update!(highlights: [ blob ])

user.highlights.count # => 2
user.highlights.first.filename # => "funky.jpg"
user.highlights.second.filename # => "town.jpg"
```

Com os padrões de configuração do Rails 6.0, atribuir a uma coleção de anexos substitui os arquivos existentes em vez de anexar a eles. Isso corresponde ao comportamento do *Active Record* ao atribuir a uma associação de coleção:

```ruby
user.highlights.attach(filename: "funky.jpg", ...)
user.highlights.count # => 1

blob = ActiveStorage::Blob.create_after_upload!(filename: "town.jpg", ...)
user.update!(highlights: [ blob ])

user.highlights.count # => 1
user.highlights.first.filename # => "town.jpg"
```

`#attach` pode ser usado para adicionar novos anexos sem remover os existentes:

```ruby
blob = ActiveStorage::Blob.create_after_upload!(filename: "town.jpg", ...)
user.highlights.attach(blob)

user.highlights.count # => 2
user.highlights.first.filename # => "funky.jpg"
user.highlights.second.filename # => "town.jpg"
```

As aplicações existentes podem aceitar este novo comportamento definindo `config.active_storage.replace_on_assign_to_many` como `true`. O comportamento antigo será descontinuado no Rails 6.1 e removido em uma versão subsequente.

Atualizando do Rails 5.1 para o Rails 5.2
-------------------------------------

Para mais informações sobre as mudanças feitas no Rails 5.2 consulte as [notas de lançamento](5_2_release_notes.html).

### *Bootsnap*

Rails 5.2 adiciona a *gem bootsnap* no [novo Gemfile](https://github.com/rails/rails/pull/29313).
O comando `app:update` o configura em `boot.rb`. Se você quiser utilizá-lo, então adicione-o no Gemfile,
caso contrário, mude o `boot.rb` para não utilizar o *bootsnap*.

### A expiração em *cookies* assinados ou criptografados está agora incorporada nos valores dos *cookies*

Para melhorar a segurança, Rails agora incorpora as informações de expiração também no valor de cookies criptografados ou assinados.

Estas novas informações incorporadas tornam estes *cookies* incompatíveis com versões do Rails mais antigas que 5.2.

Se você quer que seus cookies sejam lidos até 5.1 e anteriores, ou se ainda estiver validando seu *deploy* 5.2 e quiser permitir o *rollback* configure
 `Rails.application.config.action_dispatch.use_authenticated_cookie_encryption` para `false'.

Atualizando do Rails 5.0 para o Rails 5.1
-------------------------------------

Para mais informações sobre as mudanças feitas no Rails 5.1 consulte as [notas de lançamento](5_1_release_notes.html).

### `HashWithIndifferentAccess` de nível superior está descontinuado

Se sua aplicação usa a classe `HashWithIndifferentAccess` de nível superior, você
 deve mover lentamente seu código para usar `ActiveSupport::HashWithIndifferentAccess`.

Está apenas descontinuado, o que significa que seu código não quebrará no momento e nenhum aviso de descontinuação será exibido, mas esta constante será removida no futuro.

Além disso, se você tiver documentos *YAML* muito antigos contendo despejos (*dumps*) de tais objetos, pode ser necessário carregá-los e despejá-los novamente para ter certeza de que referenciam à constante correta, e que carregá-los não quebrará no futuro.

### `application.secrets` agora é carregado com todas as chaves como símbolos

Se sua aplicação armazena configuração aninhada em `config/secrets.yml`, todas as chaves agora são carregadas como símbolos, então o acesso usando *strings* deve ser alterado.

De:

```ruby
Rails.application.secrets[:smtp_settings]["address"]
```

Para:

```ruby
Rails.application.secrets[:smtp_settings][:address]
```

### Removido suporte obsoleto para `:text` e `:nothing` em `render`

Se suas *views* estiverem usando `render :text`, elas não funcionarão mais. O novo método de renderização de texto com o tipo MIME de `text/plain` é usar `render :plain`.

Similarmente, `render :nothing` também é removido e você deve usar o método `head` para enviar respostas que contenham apenas cabeçalhos (*headers*). Por exemplo, `head :ok` envia uma resposta 200 sem corpo (*body*) para renderizar.


Upgrading from Rails 4.2 to Rails 5.0
-------------------------------------

For more information on changes made to Rails 5.0 please see the [release notes](5_0_release_notes.html).

### Ruby 2.2.2+ required

From Ruby on Rails 5.0 onwards, Ruby 2.2.2+ is the only supported Ruby version.
Make sure you are on Ruby 2.2.2 version or greater, before you proceed.

### Active Record Models Now Inherit from ApplicationRecord by Default

In Rails 4.2, an Active Record model inherits from `ActiveRecord::Base`. In Rails 5.0,
all models inherit from `ApplicationRecord`.

`ApplicationRecord` is a new superclass for all app models, analogous to app
controllers subclassing `ApplicationController` instead of
`ActionController::Base`. This gives apps a single spot to configure app-wide
model behavior.

When upgrading from Rails 4.2 to Rails 5.0, you need to create an
`application_record.rb` file in `app/models/` and add the following content:

```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
end
```

Then make sure that all your models inherit from it.

### Halting Callback Chains via `throw(:abort)`

In Rails 4.2, when a 'before' callback returns `false` in Active Record
and Active Model, then the entire callback chain is halted. In other words,
successive 'before' callbacks are not executed, and neither is the action wrapped
in callbacks.

In Rails 5.0, returning `false` in an Active Record or Active Model callback
will not have this side effect of halting the callback chain. Instead, callback
chains must be explicitly halted by calling `throw(:abort)`.

When you upgrade from Rails 4.2 to Rails 5.0, returning `false` in those kind of
callbacks will still halt the callback chain, but you will receive a deprecation
warning about this upcoming change.

When you are ready, you can opt into the new behavior and remove the deprecation
warning by adding the following configuration to your `config/application.rb`:

```ruby
ActiveSupport.halt_callback_chains_on_return_false = false
```

Note that this option will not affect Active Support callbacks since they never
halted the chain when any value was returned.

See [#17227](https://github.com/rails/rails/pull/17227) for more details.

### ActiveJob Now Inherits from ApplicationJob by Default

In Rails 4.2, an Active Job inherits from `ActiveJob::Base`. In Rails 5.0, this
behavior has changed to now inherit from `ApplicationJob`.

When upgrading from Rails 4.2 to Rails 5.0, you need to create an
`application_job.rb` file in `app/jobs/` and add the following content:

```ruby
class ApplicationJob < ActiveJob::Base
end
```

Then make sure that all your job classes inherit from it.

See [#19034](https://github.com/rails/rails/pull/19034) for more details.

### Rails Controller Testing

#### Extraction of some helper methods to `rails-controller-testing`

`assigns` and `assert_template` have been extracted to the `rails-controller-testing` gem. To
continue using these methods in your controller tests, add `gem 'rails-controller-testing'` to
your `Gemfile`.

If you are using RSpec for testing, please see the extra configuration required in the gem's
documentation.

#### New behavior when uploading files

If you are using `ActionDispatch::Http::UploadedFile` in your tests to
upload files, you will need to change to use the similar `Rack::Test::UploadedFile`
class instead.

See [#26404](https://github.com/rails/rails/issues/26404) for more details.

### Autoloading is Disabled After Booting in the Production Environment

Autoloading is now disabled after booting in the production environment by
default.

Eager loading the application is part of the boot process, so top-level
constants are fine and are still autoloaded, no need to require their files.

Constants in deeper places only executed at runtime, like regular method bodies,
are also fine because the file defining them will have been eager loaded while booting.

For the vast majority of applications this change needs no action. But in the
very rare event that your application needs autoloading while running in
production, set `Rails.application.config.enable_dependency_loading` to true.

### XML Serialization

`ActiveModel::Serializers::Xml` has been extracted from Rails to the `activemodel-serializers-xml`
gem. To continue using XML serialization in your application, add `gem 'activemodel-serializers-xml'`
to your `Gemfile`.

### Removed Support for Legacy `mysql` Database Adapter

Rails 5 removes support for the legacy `mysql` database adapter. Most users should be able to
use `mysql2` instead. It will be converted to a separate gem when we find someone to maintain
it.

### Removed Support for Debugger

`debugger` is not supported by Ruby 2.2 which is required by Rails 5. Use `byebug` instead.

### Use `bin/rails` for running tasks and tests

Rails 5 adds the ability to run tasks and tests through `bin/rails` instead of rake. Generally
these changes are in parallel with rake, but some were ported over altogether.

To use the new test runner simply type `bin/rails test`.

`rake dev:cache` is now `bin/rails dev:cache`.

Run `bin/rails` inside your application's root directory to see the list of commands available.

### `ActionController::Parameters` No Longer Inherits from `HashWithIndifferentAccess`

Calling `params` in your application will now return an object instead of a hash. If your
parameters are already permitted, then you will not need to make any changes. If you are using `map`
and other methods that depend on being able to read the hash regardless of `permitted?` you will
need to upgrade your application to first permit and then convert to a hash.

```ruby
params.permit([:proceed_to, :return_to]).to_h
```

### `protect_from_forgery` Now Defaults to `prepend: false`

`protect_from_forgery` defaults to `prepend: false` which means that it will be inserted into
the callback chain at the point in which you call it in your application. If you want
`protect_from_forgery` to always run first, then you should change your application to use
`protect_from_forgery prepend: true`.

### Default Template Handler is Now RAW

Files without a template handler in their extension will be rendered using the raw handler.
Previously Rails would render files using the ERB template handler.

If you do not want your file to be handled via the raw handler, you should add an extension
to your file that can be parsed by the appropriate template handler.

### Added Wildcard Matching for Template Dependencies

You can now use wildcard matching for your template dependencies. For example, if you were
defining your templates as such:

```erb
<% # Template Dependency: recordings/threads/events/subscribers_changed %>
<% # Template Dependency: recordings/threads/events/completed %>
<% # Template Dependency: recordings/threads/events/uncompleted %>
```

You can now just call the dependency once with a wildcard.

```erb
<% # Template Dependency: recordings/threads/events/* %>
```

### `ActionView::Helpers::RecordTagHelper` moved to external gem (record_tag_helper)

`content_tag_for` and `div_for` have been removed in favor of just using `content_tag`. To continue using the older methods, add the `record_tag_helper` gem to your `Gemfile`:

```ruby
gem 'record_tag_helper', '~> 1.0'
```

See [#18411](https://github.com/rails/rails/pull/18411) for more details.

### Removed Support for `protected_attributes` Gem

The `protected_attributes` gem is no longer supported in Rails 5.

### Removed support for `activerecord-deprecated_finders` gem

The `activerecord-deprecated_finders` gem is no longer supported in Rails 5.

### `ActiveSupport::TestCase` Default Test Order is Now Random

When tests are run in your application, the default order is now `:random`
instead of `:sorted`. Use the following config option to set it back to `:sorted`.

```ruby
# config/environments/test.rb
Rails.application.configure do
  config.active_support.test_order = :sorted
end
```

### `ActionController::Live` became a `Concern`

If you include `ActionController::Live` in another module that is included in your controller, then you
should also extend the module with `ActiveSupport::Concern`. Alternatively, you can use the `self.included` hook
to include `ActionController::Live` directly to the controller once the `StreamingSupport` is included.

This means that if your application used to have its own streaming module, the following code
would break in production:

```ruby
# This is a work-around for streamed controllers performing authentication with Warden/Devise.
# See https://github.com/plataformatec/devise/issues/2332
# Authenticating in the router is another solution as suggested in that issue
class StreamingSupport
  include ActionController::Live # this won't work in production for Rails 5
  # extend ActiveSupport::Concern # unless you uncomment this line.

  def process(name)
    super(name)
  rescue ArgumentError => e
    if e.message == 'uncaught throw :warden'
      throw :warden
    else
      raise e
    end
  end
end
```

### New Framework Defaults

#### Active Record `belongs_to` Required by Default Option

`belongs_to` will now trigger a validation error by default if the association is not present.

This can be turned off per-association with `optional: true`.

This default will be automatically configured in new applications. If an existing application
wants to add this feature it will need to be turned on in an initializer:

```ruby
config.active_record.belongs_to_required_by_default = true
```

The configuration is by default global for all your models, but you can
override it on a per model basis. This should help you migrate all your models to have their
associations required by default.

```ruby
class Book < ApplicationRecord
  # model is not yet ready to have its association required by default

  self.belongs_to_required_by_default = false
  belongs_to(:author)
end

class Car < ApplicationRecord
  # model is ready to have its association required by default

  self.belongs_to_required_by_default = true
  belongs_to(:pilot)
end
```

#### Per-form CSRF Tokens

Rails 5 now supports per-form CSRF tokens to mitigate against code-injection attacks with forms
created by JavaScript. With this option turned on, forms in your application will each have their
own CSRF token that is specific to the action and method for that form.

```ruby
config.action_controller.per_form_csrf_tokens = true
```

#### Forgery Protection with Origin Check

You can now configure your application to check if the HTTP `Origin` header should be checked
against the site's origin as an additional CSRF defense. Set the following in your config to
true:

```ruby
config.action_controller.forgery_protection_origin_check = true
```

#### Allow Configuration of Action Mailer Queue Name

The default mailer queue name is `mailers`. This configuration option allows you to globally change
the queue name. Set the following in your config:

```ruby
config.action_mailer.deliver_later_queue_name = :new_queue_name
```

#### Support Fragment Caching in Action Mailer Views

Set `config.action_mailer.perform_caching` in your config to determine whether your Action Mailer views
should support caching.

```ruby
config.action_mailer.perform_caching = true
```

#### Configure the Output of `db:structure:dump`

If you're using `schema_search_path` or other PostgreSQL extensions, you can control how the schema is
dumped. Set to `:all` to generate all dumps, or to `:schema_search_path` to generate from schema search path.

```ruby
config.active_record.dump_schemas = :all
```

#### Configure SSL Options to Enable HSTS with Subdomains

Set the following in your config to enable HSTS when using subdomains:

```ruby
config.ssl_options = { hsts: { subdomains: true } }
```

#### Preserve Timezone of the Receiver

When using Ruby 2.4, you can preserve the timezone of the receiver when calling `to_time`.

```ruby
ActiveSupport.to_time_preserves_timezone = false
```

### Changes with JSON/JSONB serialization

In Rails 5.0, how JSON/JSONB attributes are serialized and deserialized changed. Now, if
you set a column equal to a `String`, Active Record will no longer turn that string
into a `Hash`, and will instead only return the string. This is not limited to code
interacting with models, but also affects `:default` column settings in `db/schema.rb`.
It is recommended that you do not set columns equal to a `String`, but pass a `Hash`
instead, which will be converted to and from a JSON string automatically.

Atualizando do Rails 4.1 para o Rails 4.2
-------------------------------------

### *Web Console*

Primeiro, adicione a `gem 'web-console', '~> 2.0'` ao grupo `:development` em seu `Gemfile` e execute `bundle install` (ela não foi incluída quando você atualizou o Rails). Depois de instalado, você pode simplesmente colocar uma referência ao *console helper* (ou seja, `<%= console %>`) em qualquer *view* para a qual deseja habilitá-lo. Um *console* também será fornecido em qualquer página de erro exibida em seu ambiente de desenvolvimento.

### *Responders*

Os métodos de classe `respond_with` e `respond_to` foram extraídos para a *gem* `responders`. Para usá-los, simplesmente adicione a `gem 'responders', '~> 2.0'` ao seu `Gemfile`. Chamadas para `respond_with` e `respond_to` (novamente, no nível de classe) não funcionarão mais sem incluir a *gem* `responders` em suas dependências:

```ruby
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  respond_to :html, :json

  def show
    @user = User.find(params[:id])
    respond_with @user
  end
end
```

`respond_to` em nível de instância não é afetado e não requer a *gem* adicional:

```ruby
# app/controllers/users_controller.rb

class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    respond_to do |format|
      format.html
      format.json { render json: @user }
    end
  end
end
```

Veja [#16526](https://github.com/rails/rails/pull/16526) para mais detalhes.

### Tratamento de erros em *transaction callbacks*

Atualmente, o *Active Record* suprime os erros levantados dentro de *callbacks* `after_rollback` ou `after_commit` e apenas os imprime para os logs.
Na próxima versão, esses erros não serão mais suprimidos. Em vez disso, os erros serão propagados normalmente como em outros *Active Record callbacks*.

Quando você define um *callback* `after_rollback` ou `after_commit`, você receberá um aviso de suspensão de uso sobre essa mudança futura.
Quando você estiver pronto, pode optar pelo novo comportamento e remover o aviso de suspensão de uso, adicionando a seguinte configuração ao seu
`config/application.rb`:

```ruby
config.active_record.raise_in_transactional_callbacks = true
```

Veja [#14488](https://github.com/rails/rails/pull/14488) e
[#16537](https://github.com/rails/rails/pull/16537) para mais detalhes.

### Ordenando os casos de teste

No Rails 5.0, os casos de teste serão executados em ordem aleatória por padrão. Em antecipação a esta mudança, Rails 4.2 introduziu uma nova opção de configuração
`active_support.test_order` para especificar explicitamente a ordem dos testes. Isso permite que você bloqueie o comportamento atual, definindo a opção para
`:sorted`, ou opte pelo comportamento futuro configurando a opção para `:random`.

Se você não especificar um valor para esta opção, um aviso de suspensão de uso será emitido. Para evitar isso, adicione a seguinte linha ao seu ambiente de teste:

```ruby
# config/environments/test.rb
Rails.application.configure do
  config.active_support.test_order = :sorted # ou `:random` se você preferir
end
```

### Atributos serializados

Ao usar um codificador personalizado (por exemplo, `serialize :metadata, JSON`), atribuir `nil` a um atributo serializado irá salvá-lo no banco de dados
como `NULL` em vez de passar o valor `nil` através do codificador (por exemplo, `"null"` quando usando o codificador `JSON`).

### Nível de *log* em produção

No Rails 5, o nível de *log* padrão para o ambiente de produção será alterado para `:debug` (de `:info`). Para preservar o padrão atual, adicione a seguinte linha para o seu `production.rb`:

```ruby
# Defina como `:info` para corresponder ao padrão atual, ou defina como `:debug` para ativar o padrão futuro.
config.log_level = :info
```

### `after_bundle` em Rails *templates*

Se você tem um *Rails template* que adiciona todos os arquivos no controle de versão, isso falhará ao adicionar os *binstubs* gerados porque ele é executado antes do Bundler:

```ruby
# template.rb
generate(:scaffold, "person name:string")
route "root to: 'people#index'"
rake("db:migrate")

git :init
git add: "."
git commit: %Q{ -m 'Initial commit' }
```

Agora você pode envolver as chamadas `git` em um bloco `after_bundle`. Isso será executado depois que os *binstubs* foram gerados.

```ruby
# template.rb
generate(:scaffold, "person name:string")
route "root to: 'people#index'"
rake("db:migrate")

after_bundle do
  git :init
  git add: "."
  git commit: %Q{ -m 'Initial commit' }
end
```

### Rails *HTML Sanitizer*

Há uma nova opção para sanitizar fragmentos de HTML em suas aplicações. A venerável abordagem *html-scanner* agora está oficialmente sendo descontinuada em favor de
[`Rails HTML Sanitizer`](https://github.com/rails/rails-html-sanitizer).

Isso significa que os métodos `sanitize`, `sanitize_css`, `strip_tags` e `strip_links` são apoiados por uma nova implementação.

Este novo *sanitizer* usa internamente [Loofah](https://github.com/flavorjones/loofah). Loofah, por sua vez, usa Nokogiri, que
envolve analisadores XML escritos em C e Java, portanto, a sanitização deve ser mais rápida não importa qual versão do Ruby você execute.

A nova versão atualiza `sanitize`, então pode usar um `Loofah::Scrubber` para uma depuração poderosa.
[Veja alguns exemplos de depuradores aqui](https://github.com/flavorjones/loofah#loofahscrubber).

Dois novos depuradores também foram adicionados: `PermitScrubber` e `TargetScrubber`.
Leia o [*gem's readme*](https://github.com/rails/rails-html-sanitizer) para mais informações.

A documentação para `PermitScrubber` e `TargetScrubber` explica como você pode obter controle total sobre quando e como os elementos devem ser removidos.

Se sua aplicação precisa usar a implementação antiga do *sanitizer*, inclua `rails-deprecated_sanitizer` em seu `Gemfile`:

```ruby
gem 'rails-deprecated_sanitizer'
```

### Testando *Rails DOM*

O [módulo `TagAssertions`](https://api.rubyonrails.org/v4.1/classes/ActionDispatch/Assertions/TagAssertions.html) (contendo métodos como `assert_tag`), [foi descontinuado](https://github.com/rails/rails/blob/6061472b8c310158a2a2e8e9a6b81a1aef6b60fe/actionpack/lib/action_dispatch/testing/assertions/dom.rb) em favor dos métodos `assert_select` do módulo `SelectorAssertions`, que foi extraído para a [*gem rails-dom-testing*](https://github.com/rails/rails-dom-testing).

### Tokens de autenticidade mascarados

A fim de mitigar ataques SSL, `form_authenticity_token` agora é mascarado para que varie com cada solicitação (*request*). Assim, os *tokens* são validados desmascarando e depois descriptografando. Como resultado, quaisquer estratégias para verificar solicitações de formulários não-rails que dependiam de um *token* CSRF de sessão estática devem levar isso em consideração.

### *Action Mailer*

Anteriormente, chamar um método *mailer* em uma classe *mailer* resultaria no método de instância correspondente sendo executado diretamente. Com a introdução de
*Active Job* e `#deliver_later`, isso não é mais verdade. No Rails 4.2, a invocação dos métodos de instância é adiada até `deliver_now` ou
`deliver_later` sejam chamados. Por exemplo:

```ruby
class Notifier < ActionMailer::Base
  def notify(user, ...)
    puts "Called"
    mail(to: user.email, ...)
  end
end
```

```ruby
mail = Notifier.notify(user, ...) # Notifier#notify ainda não é chamado neste momento
mail = mail.deliver_now           # Imprime "Called"
```

Isso não deve resultar em diferenças perceptíveis para a maioria das aplicações.
No entanto, se você precisar que alguns métodos não-*mailer* sejam executados de forma síncrona, e
você estava contando anteriormente com o comportamento de *proxy* síncrono, você deve
definí-los como métodos de classe na classe *mailer* diretamente:

```ruby
class Notifier < ActionMailer::Base
  def self.broadcast_notifications(users, ...)
    users.each { |user| Notifier.notify(user, ...) }
  end
end
```

### Suporte para chave estrangeira

A migração DSL foi expandida para suportar definições de chave estrangeira. Se
você tem usado a *gem Foreigner*, você pode querer considerar removê-la.
Observe que o suporte de chave estrangeira do Rails é um subconjunto de *Foreigner*. Isso significa
que nem todas as definições *Foreigner* podem ser totalmente substituídas pela contraparte
DSL de migração Rails.

O procedimento de migração é o seguinte:

1. remova `gem "foreigner"` do `Gemfile`.
2. execute `bundle install`.
3. execute `bin/rake db:schema:dump`.
4. certifique-se de que `db/schema.rb` contém todas as definições de chave estrangeira com
as opções necessárias.

Upgrading from Rails 4.0 to Rails 4.1
-------------------------------------

### CSRF protection from remote `<script>` tags

Or, "whaaat my tests are failing!!!?" or "my `<script>` widget is busted!!"

Cross-site request forgery (CSRF) protection now covers GET requests with
JavaScript responses, too. This prevents a third-party site from remotely
referencing your JavaScript with a `<script>` tag to extract sensitive data.

This means that your functional and integration tests that use

```ruby
get :index, format: :js
```

will now trigger CSRF protection. Switch to

```ruby
xhr :get, :index, format: :js
```

to explicitly test an `XmlHttpRequest`.

NOTE: Your own `<script>` tags are treated as cross-origin and blocked by
default, too. If you really mean to load JavaScript from `<script>` tags,
you must now explicitly skip CSRF protection on those actions.

### Spring

If you want to use Spring as your application preloader you need to:

1. Add `gem 'spring', group: :development` to your `Gemfile`.
2. Install spring using `bundle install`.
3. Generate the Spring binstub with `bundle exec spring binstub`.

NOTE: User defined rake tasks will run in the `development` environment by
default. If you want them to run in other environments consult the
[Spring README](https://github.com/rails/spring#rake).

### `config/secrets.yml`

If you want to use the new `secrets.yml` convention to store your application's
secrets, you need to:

1. Create a `secrets.yml` file in your `config` folder with the following content:

    ```yaml
    development:
      secret_key_base:

    test:
      secret_key_base:

    production:
      secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
    ```

2. Use your existing `secret_key_base` from the `secret_token.rb` initializer to
   set the `SECRET_KEY_BASE` environment variable for whichever users running the
   Rails application in production. Alternatively, you can simply copy the existing
   `secret_key_base` from the `secret_token.rb` initializer to `secrets.yml`
   under the `production` section, replacing `<%= ENV["SECRET_KEY_BASE"] %>`.

3. Remove the `secret_token.rb` initializer.

4. Use `rake secret` to generate new keys for the `development` and `test` sections.

5. Restart your server.

### Changes to test helper

If your test helper contains a call to
`ActiveRecord::Migration.check_pending!` this can be removed. The check
is now done automatically when you `require "rails/test_help"`, although
leaving this line in your helper is not harmful in any way.

### Cookies serializer

Applications created before Rails 4.1 uses `Marshal` to serialize cookie values into
the signed and encrypted cookie jars. If you want to use the new `JSON`-based format
in your application, you can add an initializer file with the following content:

```ruby
Rails.application.config.action_dispatch.cookies_serializer = :hybrid
```

This would transparently migrate your existing `Marshal`-serialized cookies into the
new `JSON`-based format.

When using the `:json` or `:hybrid` serializer, you should beware that not all
Ruby objects can be serialized as JSON. For example, `Date` and `Time` objects
will be serialized as strings, and `Hash`es will have their keys stringified.

```ruby
class CookiesController < ApplicationController
  def set_cookie
    cookies.encrypted[:expiration_date] = Date.tomorrow # => Thu, 20 Mar 2014
    redirect_to action: 'read_cookie'
  end

  def read_cookie
    cookies.encrypted[:expiration_date] # => "2014-03-20"
  end
end
```

It's advisable that you only store simple data (strings and numbers) in cookies.
If you have to store complex objects, you would need to handle the conversion
manually when reading the values on subsequent requests.

If you use the cookie session store, this would apply to the `session` and
`flash` hash as well.

### Flash structure changes

Flash message keys are
[normalized to strings](https://github.com/rails/rails/commit/a668beffd64106a1e1fedb71cc25eaaa11baf0c1). They
can still be accessed using either symbols or strings. Looping through the flash
will always yield string keys:

```ruby
flash["string"] = "a string"
flash[:symbol] = "a symbol"

# Rails < 4.1
flash.keys # => ["string", :symbol]

# Rails >= 4.1
flash.keys # => ["string", "symbol"]
```

Make sure you are comparing Flash message keys against strings.

### Changes in JSON handling

There are a few major changes related to JSON handling in Rails 4.1.

#### MultiJSON removal

MultiJSON has reached its [end-of-life](https://github.com/rails/rails/pull/10576)
and has been removed from Rails.

If your application currently depends on MultiJSON directly, you have a few options:

1. Add 'multi_json' to your `Gemfile`. Note that this might cease to work in the future

2. Migrate away from MultiJSON by using `obj.to_json`, and `JSON.parse(str)` instead.

WARNING: Do not simply replace `MultiJson.dump` and `MultiJson.load` with
`JSON.dump` and `JSON.load`. These JSON gem APIs are meant for serializing and
deserializing arbitrary Ruby objects and are generally [unsafe](https://ruby-doc.org/stdlib-2.2.2/libdoc/json/rdoc/JSON.html#method-i-load).

#### JSON gem compatibility

Historically, Rails had some compatibility issues with the JSON gem. Using
`JSON.generate` and `JSON.dump` inside a Rails application could produce
unexpected errors.

Rails 4.1 fixed these issues by isolating its own encoder from the JSON gem. The
JSON gem APIs will function as normal, but they will not have access to any
Rails-specific features. For example:

```ruby
class FooBar
  def as_json(options = nil)
    { foo: 'bar' }
  end
end
```

```irb
irb> FooBar.new.to_json
=> "{\"foo\":\"bar\"}"
irb> JSON.generate(FooBar.new, quirks_mode: true)
=> "\"#<FooBar:0x007fa80a481610>\""
```

#### New JSON encoder

The JSON encoder in Rails 4.1 has been rewritten to take advantage of the JSON
gem. For most applications, this should be a transparent change. However, as
part of the rewrite, the following features have been removed from the encoder:

1. Circular data structure detection
2. Support for the `encode_json` hook
3. Option to encode `BigDecimal` objects as numbers instead of strings

If your application depends on one of these features, you can get them back by
adding the [`activesupport-json_encoder`](https://github.com/rails/activesupport-json_encoder)
gem to your `Gemfile`.

#### JSON representation of Time objects

`#as_json` for objects with time component (`Time`, `DateTime`, `ActiveSupport::TimeWithZone`)
now returns millisecond precision by default. If you need to keep old behavior with no millisecond
precision, set the following in an initializer:

```ruby
ActiveSupport::JSON::Encoding.time_precision = 0
```

### Usage of `return` within inline callback blocks

Previously, Rails allowed inline callback blocks to use `return` this way:

```ruby
class ReadOnlyModel < ActiveRecord::Base
  before_save { return false } # BAD
end
```

This behavior was never intentionally supported. Due to a change in the internals
of `ActiveSupport::Callbacks`, this is no longer allowed in Rails 4.1. Using a
`return` statement in an inline callback block causes a `LocalJumpError` to
be raised when the callback is executed.

Inline callback blocks using `return` can be refactored to evaluate to the
returned value:

```ruby
class ReadOnlyModel < ActiveRecord::Base
  before_save { false } # GOOD
end
```

Alternatively, if `return` is preferred it is recommended to explicitly define
a method:

```ruby
class ReadOnlyModel < ActiveRecord::Base
  before_save :before_save_callback # GOOD

  private
    def before_save_callback
      return false
    end
end
```

This change applies to most places in Rails where callbacks are used, including
Active Record and Active Model callbacks, as well as filters in Action
Controller (e.g. `before_action`).

See [this pull request](https://github.com/rails/rails/pull/13271) for more
details.

### Methods defined in Active Record fixtures

Rails 4.1 evaluates each fixture's ERB in a separate context, so helper methods
defined in a fixture will not be available in other fixtures.

Helper methods that are used in multiple fixtures should be defined on modules
included in the newly introduced `ActiveRecord::FixtureSet.context_class`, in
`test_helper.rb`.

```ruby
module FixtureFileHelpers
  def file_sha(path)
    Digest::SHA2.hexdigest(File.read(Rails.root.join('test/fixtures', path)))
  end
end

ActiveRecord::FixtureSet.context_class.include FixtureFileHelpers
```

### I18n enforcing available locales

Rails 4.1 now defaults the I18n option `enforce_available_locales` to `true`. This
means that it will make sure that all locales passed to it must be declared in
the `available_locales` list.

To disable it (and allow I18n to accept *any* locale option) add the following
configuration to your application:

```ruby
config.i18n.enforce_available_locales = false
```

Note that this option was added as a security measure, to ensure user input
cannot be used as locale information unless it is previously known. Therefore,
it's recommended not to disable this option unless you have a strong reason for
doing so.

### Mutator methods called on Relation

`Relation` no longer has mutator methods like `#map!` and `#delete_if`. Convert
to an `Array` by calling `#to_a` before using these methods.

It intends to prevent odd bugs and confusion in code that call mutator
methods directly on the `Relation`.

```ruby
# Instead of this
Author.where(name: 'Hank Moody').compact!

# Now you have to do this
authors = Author.where(name: 'Hank Moody').to_a
authors.compact!
```

### Changes on Default Scopes

Default scopes are no longer overridden by chained conditions.

In previous versions when you defined a `default_scope` in a model
it was overridden by chained conditions in the same field. Now it
is merged like any other scope.

Before:

```ruby
class User < ActiveRecord::Base
  default_scope { where state: 'pending' }
  scope :active, -> { where state: 'active' }
  scope :inactive, -> { where state: 'inactive' }
end

User.all
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'

User.active
# SELECT "users".* FROM "users" WHERE "users"."state" = 'active'

User.where(state: 'inactive')
# SELECT "users".* FROM "users" WHERE "users"."state" = 'inactive'
```

After:

```ruby
class User < ActiveRecord::Base
  default_scope { where state: 'pending' }
  scope :active, -> { where state: 'active' }
  scope :inactive, -> { where state: 'inactive' }
end

User.all
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'

User.active
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending' AND "users"."state" = 'active'

User.where(state: 'inactive')
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending' AND "users"."state" = 'inactive'
```

To get the previous behavior it is needed to explicitly remove the
`default_scope` condition using `unscoped`, `unscope`, `rewhere` or
`except`.

```ruby
class User < ActiveRecord::Base
  default_scope { where state: 'pending' }
  scope :active, -> { unscope(where: :state).where(state: 'active') }
  scope :inactive, -> { rewhere state: 'inactive' }
end

User.all
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'

User.active
# SELECT "users".* FROM "users" WHERE "users"."state" = 'active'

User.inactive
# SELECT "users".* FROM "users" WHERE "users"."state" = 'inactive'
```

### Rendering content from string

Rails 4.1 introduces `:plain`, `:html`, and `:body` options to `render`. Those
options are now the preferred way to render string-based content, as it allows
you to specify which content type you want the response sent as.

* `render :plain` will set the content type to `text/plain`
* `render :html` will set the content type to `text/html`
* `render :body` will *not* set the content type header.

From the security standpoint, if you don't expect to have any markup in your
response body, you should be using `render :plain` as most browsers will escape
unsafe content in the response for you.

We will be deprecating the use of `render :text` in a future version. So please
start using the more precise `:plain`, `:html`, and `:body` options instead.
Using `render :text` may pose a security risk, as the content is sent as
`text/html`.

### PostgreSQL json and hstore datatypes

Rails 4.1 will map `json` and `hstore` columns to a string-keyed Ruby `Hash`.
In earlier versions, a `HashWithIndifferentAccess` was used. This means that
symbol access is no longer supported. This is also the case for
`store_accessors` based on top of `json` or `hstore` columns. Make sure to use
string keys consistently.

### Explicit block use for `ActiveSupport::Callbacks`

Rails 4.1 now expects an explicit block to be passed when calling
`ActiveSupport::Callbacks.set_callback`. This change stems from
`ActiveSupport::Callbacks` being largely rewritten for the 4.1 release.

```ruby
# Previously in Rails 4.0
set_callback :save, :around, ->(r, &block) { stuff; result = block.call; stuff }

# Now in Rails 4.1
set_callback :save, :around, ->(r, block) { stuff; result = block.call; stuff }
```

Upgrading from Rails 3.2 to Rails 4.0
-------------------------------------

If your application is currently on any version of Rails older than 3.2.x, you should upgrade to Rails 3.2 before attempting one to Rails 4.0.

The following changes are meant for upgrading your application to Rails 4.0.

### HTTP PATCH

Rails 4 now uses `PATCH` as the primary HTTP verb for updates when a RESTful
resource is declared in `config/routes.rb`. The `update` action is still used,
and `PUT` requests will continue to be routed to the `update` action as well.
So, if you're using only the standard RESTful routes, no changes need to be made:

```ruby
resources :users
```

```erb
<%= form_for @user do |f| %>
```

```ruby
class UsersController < ApplicationController
  def update
    # No change needed; PATCH will be preferred, and PUT will still work.
  end
end
```

However, you will need to make a change if you are using `form_for` to update
a resource in conjunction with a custom route using the `PUT` HTTP method:

```ruby
resources :users do
  put :update_name, on: :member
end
```

```erb
<%= form_for [ :update_name, @user ] do |f| %>
```

```ruby
class UsersController < ApplicationController
  def update_name
    # Change needed; form_for will try to use a non-existent PATCH route.
  end
end
```

If the action is not being used in a public API and you are free to change the
HTTP method, you can update your route to use `patch` instead of `put`:

```ruby
resources :users do
  patch :update_name, on: :member
end
```

`PUT` requests to `/users/:id` in Rails 4 get routed to `update` as they are
today. So, if you have an API that gets real PUT requests it is going to work.
The router also routes `PATCH` requests to `/users/:id` to the `update` action.

If the action is being used in a public API and you can't change to HTTP method
being used, you can update your form to use the `PUT` method instead:

```erb
<%= form_for [ :update_name, @user ], method: :put do |f| %>
```

For more on PATCH and why this change was made, see [this post](https://weblog.rubyonrails.org/2012/2/26/edge-rails-patch-is-the-new-primary-http-method-for-updates/)
on the Rails blog.

#### A note about media types

The errata for the `PATCH` verb [specifies that a 'diff' media type should be
used with `PATCH`](http://www.rfc-editor.org/errata_search.php?rfc=5789). One
such format is [JSON Patch](https://tools.ietf.org/html/rfc6902). While Rails
does not support JSON Patch natively, it's easy enough to add support:

```ruby
# in your controller:
def update
  respond_to do |format|
    format.json do
      # perform a partial update
      @article.update params[:article]
    end

    format.json_patch do
      # perform sophisticated change
    end
  end
end
```

```ruby
# config/initializers/json_patch.rb
Mime::Type.register 'application/json-patch+json', :json_patch
```

As JSON Patch was only recently made into an RFC, there aren't a lot of great
Ruby libraries yet. Aaron Patterson's
[hana](https://github.com/tenderlove/hana) is one such gem, but doesn't have
full support for the last few changes in the specification.

### Gemfile

Rails 4.0 removed the `assets` group from `Gemfile`. You'd need to remove that
line from your `Gemfile` when upgrading. You should also update your application
file (in `config/application.rb`):

```ruby
# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)
```

### vendor/plugins

Rails 4.0 no longer supports loading plugins from `vendor/plugins`. You must replace any plugins by extracting them to gems and adding them to your `Gemfile`. If you choose not to make them gems, you can move them into, say, `lib/my_plugin/*` and add an appropriate initializer in `config/initializers/my_plugin.rb`.

### Active Record

* Rails 4.0 has removed the identity map from Active Record, due to [some inconsistencies with associations](https://github.com/rails/rails/commit/302c912bf6bcd0fa200d964ec2dc4a44abe328a6). If you have manually enabled it in your application, you will have to remove the following config that has no effect anymore: `config.active_record.identity_map`.

* The `delete` method in collection associations can now receive `Integer` or `String` arguments as record ids, besides records, pretty much like the `destroy` method does. Previously it raised `ActiveRecord::AssociationTypeMismatch` for such arguments. From Rails 4.0 on `delete` automatically tries to find the records matching the given ids before deleting them.

* In Rails 4.0 when a column or a table is renamed the related indexes are also renamed. If you have migrations which rename the indexes, they are no longer needed.

* Rails 4.0 has changed `serialized_attributes` and `attr_readonly` to class methods only. You shouldn't use instance methods since it's now deprecated. You should change them to use class methods, e.g. `self.serialized_attributes` to `self.class.serialized_attributes`.

* When using the default coder, assigning `nil` to a serialized attribute will save it
to the database as `NULL` instead of passing the `nil` value through YAML (`"--- \n...\n"`).

* Rails 4.0 has removed `attr_accessible` and `attr_protected` feature in favor of Strong Parameters. You can use the [Protected Attributes gem](https://github.com/rails/protected_attributes) for a smooth upgrade path.

* If you are not using Protected Attributes, you can remove any options related to
this gem such as `whitelist_attributes` or `mass_assignment_sanitizer` options.

* Rails 4.0 requires that scopes use a callable object such as a Proc or lambda:

    ```ruby
      scope :active, where(active: true)

      # becomes
      scope :active, -> { where active: true }
    ```

* Rails 4.0 has deprecated `ActiveRecord::Fixtures` in favor of `ActiveRecord::FixtureSet`.

* Rails 4.0 has deprecated `ActiveRecord::TestCase` in favor of `ActiveSupport::TestCase`.

* Rails 4.0 has deprecated the old-style hash based finder API. This means that
  methods which previously accepted "finder options" no longer do.  For example, `Book.find(:all, conditions: { name: '1984' })` has been deprecated in favor of `Book.where(name: '1984')`

* All dynamic methods except for `find_by_...` and `find_by_...!` are deprecated.
  Here's how you can handle the changes:

      * `find_all_by_...`           becomes `where(...)`.
      * `find_last_by_...`          becomes `where(...).last`.
      * `scoped_by_...`             becomes `where(...)`.
      * `find_or_initialize_by_...` becomes `find_or_initialize_by(...)`.
      * `find_or_create_by_...`     becomes `find_or_create_by(...)`.

* Note that `where(...)` returns a relation, not an array like the old finders. If you require an `Array`, use `where(...).to_a`.

* These equivalent methods may not execute the same SQL as the previous implementation.

* To re-enable the old finders, you can use the [activerecord-deprecated_finders gem](https://github.com/rails/activerecord-deprecated_finders).

* Rails 4.0 has changed to default join table for `has_and_belongs_to_many` relations to strip the common prefix off the second table name. Any existing `has_and_belongs_to_many` relationship between models with a common prefix must be specified with the `join_table` option. For example:

    ```ruby
    CatalogCategory < ActiveRecord::Base
      has_and_belongs_to_many :catalog_products, join_table: 'catalog_categories_catalog_products'
    end

    CatalogProduct < ActiveRecord::Base
      has_and_belongs_to_many :catalog_categories, join_table: 'catalog_categories_catalog_products'
    end
    ```

* Note that the prefix takes scopes into account as well, so relations between `Catalog::Category` and `Catalog::Product` or `Catalog::Category` and `CatalogProduct` need to be updated similarly.

### Active Resource

Rails 4.0 extracted Active Resource to its own gem. If you still need the feature you can add the [Active Resource gem](https://github.com/rails/activeresource) in your `Gemfile`.

### Active Model

* Rails 4.0 has changed how errors attach with the `ActiveModel::Validations::ConfirmationValidator`. Now when confirmation validations fail, the error will be attached to `:#{attribute}_confirmation` instead of `attribute`.

* Rails 4.0 has changed `ActiveModel::Serializers::JSON.include_root_in_json` default value to `false`. Now, Active Model Serializers and Active Record objects have the same default behavior. This means that you can comment or remove the following option in the `config/initializers/wrap_parameters.rb` file:

    ```ruby
    # Disable root element in JSON by default.
    # ActiveSupport.on_load(:active_record) do
    #   self.include_root_in_json = false
    # end
    ```

### Action Pack

*   Rails 4.0 introduces `ActiveSupport::KeyGenerator` and uses this as a base from which to generate and verify signed cookies (among other things). Existing signed cookies generated with Rails 3.x will be transparently upgraded if you leave your existing `secret_token` in place and add the new `secret_key_base`.

    ```ruby
      # config/initializers/secret_token.rb
      Myapp::Application.config.secret_token = 'existing secret token'
      Myapp::Application.config.secret_key_base = 'new secret key base'
    ```

    Please note that you should wait to set `secret_key_base` until you have 100% of your userbase on Rails 4.x and are reasonably sure you will not need to rollback to Rails 3.x. This is because cookies signed based on the new `secret_key_base` in Rails 4.x are not backwards compatible with Rails 3.x. You are free to leave your existing `secret_token` in place, not set the new `secret_key_base`, and ignore the deprecation warnings until you are reasonably sure that your upgrade is otherwise complete.

    If you are relying on the ability for external applications or JavaScript to be able to read your Rails app's signed session cookies (or signed cookies in general) you should not set `secret_key_base` until you have decoupled these concerns.

*   Rails 4.0 encrypts the contents of cookie-based sessions if `secret_key_base` has been set. Rails 3.x signed, but did not encrypt, the contents of cookie-based session. Signed cookies are "secure" in that they are verified to have been generated by your app and are tamper-proof. However, the contents can be viewed by end users, and encrypting the contents eliminates this caveat/concern without a significant performance penalty.

    Please read [Pull Request #9978](https://github.com/rails/rails/pull/9978) for details on the move to encrypted session cookies.

* Rails 4.0 removed the `ActionController::Base.asset_path` option. Use the assets pipeline feature.

* Rails 4.0 has deprecated `ActionController::Base.page_cache_extension` option. Use `ActionController::Base.default_static_extension` instead.

* Rails 4.0 has removed Action and Page caching from Action Pack. You will need to add the `actionpack-action_caching` gem in order to use `caches_action` and the `actionpack-page_caching` to use `caches_page` in your controllers.

* Rails 4.0 has removed the XML parameters parser. You will need to add the `actionpack-xml_parser` gem if you require this feature.

* Rails 4.0 changes the default `layout` lookup set using symbols or procs that return nil. To get the "no layout" behavior, return false instead of nil.

* Rails 4.0 changes the default memcached client from `memcache-client` to `dalli`. To upgrade, simply add `gem 'dalli'` to your `Gemfile`.

* Rails 4.0 deprecates the `dom_id` and `dom_class` methods in controllers (they are fine in views). You will need to include the `ActionView::RecordIdentifier` module in controllers requiring this feature.

* Rails 4.0 deprecates the `:confirm` option for the `link_to` helper. You should
instead rely on a data attribute (e.g. `data: { confirm: 'Are you sure?' }`).
This deprecation also concerns the helpers based on this one (such as `link_to_if`
or `link_to_unless`).

* Rails 4.0 changed how `assert_generates`, `assert_recognizes`, and `assert_routing` work. Now all these assertions raise `Assertion` instead of `ActionController::RoutingError`.

*   Rails 4.0 raises an `ArgumentError` if clashing named routes are defined. This can be triggered by explicitly defined named routes or by the `resources` method. Here are two examples that clash with routes named `example_path`:

    ```ruby
    get 'one' => 'test#example', as: :example
    get 'two' => 'test#example', as: :example
    ```

    ```ruby
    resources :examples
    get 'clashing/:id' => 'test#example', as: :example
    ```

    In the first case, you can simply avoid using the same name for multiple
    routes. In the second, you can use the `only` or `except` options provided by
    the `resources` method to restrict the routes created as detailed in the
    [Routing Guide](routing.html#restricting-the-routes-created).

*   Rails 4.0 also changed the way unicode character routes are drawn. Now you can draw unicode character routes directly. If you already draw such routes, you must change them, for example:

    ```ruby
    get Rack::Utils.escape('こんにちは'), controller: 'welcome', action: 'index'
    ```

    becomes

    ```ruby
    get 'こんにちは', controller: 'welcome', action: 'index'
    ```

*   Rails 4.0 requires that routes using `match` must specify the request method. For example:

    ```ruby
      # Rails 3.x
      match '/' => 'root#index'

      # becomes
      match '/' => 'root#index', via: :get

      # or
      get '/' => 'root#index'
    ```

*   Rails 4.0 has removed `ActionDispatch::BestStandardsSupport` middleware, `<!DOCTYPE html>` already triggers standards mode per https://msdn.microsoft.com/en-us/library/jj676915(v=vs.85).aspx and ChromeFrame header has been moved to `config.action_dispatch.default_headers`.

    Remember you must also remove any references to the middleware from your application code, for example:

    ```ruby
    # Raise exception
    config.middleware.insert_before(Rack::Lock, ActionDispatch::BestStandardsSupport)
    ```

    Also check your environment settings for `config.action_dispatch.best_standards_support` and remove it if present.

*   Rails 4.0 allows configuration of HTTP headers by setting `config.action_dispatch.default_headers`. The defaults are as follows:

    ```ruby
      config.action_dispatch.default_headers = {
        'X-Frame-Options' => 'SAMEORIGIN',
        'X-XSS-Protection' => '1; mode=block'
      }
    ```

    Please note that if your application is dependent on loading certain pages in a `<frame>` or `<iframe>`, then you may need to explicitly set `X-Frame-Options` to `ALLOW-FROM ...` or `ALLOWALL`.

* In Rails 4.0, precompiling assets no longer automatically copies non-JS/CSS assets from `vendor/assets` and `lib/assets`. Rails application and engine developers should put these assets in `app/assets` or configure `config.assets.precompile`.

* In Rails 4.0, `ActionController::UnknownFormat` is raised when the action doesn't handle the request format. By default, the exception is handled by responding with 406 Not Acceptable, but you can override that now. In Rails 3, 406 Not Acceptable was always returned. No overrides.

* In Rails 4.0, a generic `ActionDispatch::ParamsParser::ParseError` exception is raised when `ParamsParser` fails to parse request params. You will want to rescue this exception instead of the low-level `MultiJson::DecodeError`, for example.

* In Rails 4.0, `SCRIPT_NAME` is properly nested when engines are mounted on an app that's served from a URL prefix. You no longer have to set `default_url_options[:script_name]` to work around overwritten URL prefixes.

* Rails 4.0 deprecated `ActionController::Integration` in favor of `ActionDispatch::Integration`.
* Rails 4.0 deprecated `ActionController::IntegrationTest` in favor of `ActionDispatch::IntegrationTest`.
* Rails 4.0 deprecated `ActionController::PerformanceTest` in favor of `ActionDispatch::PerformanceTest`.
* Rails 4.0 deprecated `ActionController::AbstractRequest` in favor of `ActionDispatch::Request`.
* Rails 4.0 deprecated `ActionController::Request` in favor of `ActionDispatch::Request`.
* Rails 4.0 deprecated `ActionController::AbstractResponse` in favor of `ActionDispatch::Response`.
* Rails 4.0 deprecated `ActionController::Response` in favor of `ActionDispatch::Response`.
* Rails 4.0 deprecated `ActionController::Routing` in favor of `ActionDispatch::Routing`.

### Active Support

Rails 4.0 removes the `j` alias for `ERB::Util#json_escape` since `j` is already used for `ActionView::Helpers::JavaScriptHelper#escape_javascript`.

#### Cache

The caching method changed between Rails 3.x and 4.0. You should [change the cache namespace](https://guides.rubyonrails.org/caching_with_rails.html#activesupport-cache-store) and roll out with a cold cache.

### Helpers Loading Order

The order in which helpers from more than one directory are loaded has changed in Rails 4.0. Previously, they were gathered and then sorted alphabetically. After upgrading to Rails 4.0, helpers will preserve the order of loaded directories and will be sorted alphabetically only within each directory. Unless you explicitly use the `helpers_path` parameter, this change will only impact the way of loading helpers from engines. If you rely on the ordering, you should check if correct methods are available after upgrade. If you would like to change the order in which engines are loaded, you can use `config.railties_order=` method.

### Active Record Observer and Action Controller Sweeper

`ActiveRecord::Observer` and `ActionController::Caching::Sweeper` have been extracted to the `rails-observers` gem. You will need to add the `rails-observers` gem if you require these features.

### sprockets-rails

* `assets:precompile:primary` and `assets:precompile:all` have been removed. Use `assets:precompile` instead.
* The `config.assets.compress` option should be changed to `config.assets.js_compressor` like so for instance:

    ```ruby
    config.assets.js_compressor = :uglifier
    ```

### sass-rails

* `asset-url` with two arguments is deprecated. For example: `asset-url("rails.png", image)` becomes `asset-url("rails.png")`.

Upgrading from Rails 3.1 to Rails 3.2
-------------------------------------

If your application is currently on any version of Rails older than 3.1.x, you
should upgrade to Rails 3.1 before attempting an update to Rails 3.2.

The following changes are meant for upgrading your application to the latest
3.2.x version of Rails.

### Gemfile

Make the following changes to your `Gemfile`.

```ruby
gem 'rails', '3.2.21'

group :assets do
  gem 'sass-rails',   '~> 3.2.6'
  gem 'coffee-rails', '~> 3.2.2'
  gem 'uglifier',     '>= 1.0.3'
end
```

### config/environments/development.rb

There are a couple of new configuration settings that you should add to your development environment:

```ruby
# Raise exception on mass assignment protection for Active Record models
config.active_record.mass_assignment_sanitizer = :strict

# Log the query plan for queries taking more than this (works
# with SQLite, MySQL, and PostgreSQL)
config.active_record.auto_explain_threshold_in_seconds = 0.5
```

### config/environments/test.rb

The `mass_assignment_sanitizer` configuration setting should also be added to `config/environments/test.rb`:

```ruby
# Raise exception on mass assignment protection for Active Record models
config.active_record.mass_assignment_sanitizer = :strict
```

### vendor/plugins

Rails 3.2 deprecates `vendor/plugins` and Rails 4.0 will remove them completely. While it's not strictly necessary as part of a Rails 3.2 upgrade, you can start replacing any plugins by extracting them to gems and adding them to your `Gemfile`. If you choose not to make them gems, you can move them into, say, `lib/my_plugin/*` and add an appropriate initializer in `config/initializers/my_plugin.rb`.

### Active Record

Option `:dependent => :restrict` has been removed from `belongs_to`. If you want to prevent deleting the object if there are any associated objects, you can set `:dependent => :destroy` and return `false` after checking for existence of association from any of the associated object's destroy callbacks.

Upgrading from Rails 3.0 to Rails 3.1
-------------------------------------

If your application is currently on any version of Rails older than 3.0.x, you should upgrade to Rails 3.0 before attempting an update to Rails 3.1.

The following changes are meant for upgrading your application to Rails 3.1.12, the last 3.1.x version of Rails.

### Gemfile

Make the following changes to your `Gemfile`.

```ruby
gem 'rails', '3.1.12'
gem 'mysql2'

# Needed for the new asset pipeline
group :assets do
  gem 'sass-rails',   '~> 3.1.7'
  gem 'coffee-rails', '~> 3.1.1'
  gem 'uglifier',     '>= 1.0.3'
end

# jQuery is the default JavaScript library in Rails 3.1
gem 'jquery-rails'
```

### config/application.rb

The asset pipeline requires the following additions:

```ruby
config.assets.enabled = true
config.assets.version = '1.0'
```

If your application is using an "/assets" route for a resource you may want to change the prefix used for assets to avoid conflicts:

```ruby
# Defaults to '/assets'
config.assets.prefix = '/asset-files'
```

### config/environments/development.rb

Remove the RJS setting `config.action_view.debug_rjs = true`.

Add these settings if you enable the asset pipeline:

```ruby
# Do not compress assets
config.assets.compress = false

# Expands the lines which load the assets
config.assets.debug = true
```

### config/environments/production.rb

Again, most of the changes below are for the asset pipeline. You can read more about these in the [Asset Pipeline](asset_pipeline.html) guide.

```ruby
# Compress JavaScripts and CSS
config.assets.compress = true

# Don't fallback to assets pipeline if a precompiled asset is missed
config.assets.compile = false

# Generate digests for assets URLs
config.assets.digest = true

# Defaults to Rails.root.join("public/assets")
# config.assets.manifest = YOUR_PATH

# Precompile additional assets (application.js, application.css, and all non-JS/CSS are already added)
# config.assets.precompile += %w( admin.js admin.css )

# Force all access to the app over SSL, use Strict-Transport-Security, and use secure cookies.
# config.force_ssl = true
```

### config/environments/test.rb

You can help test performance with these additions to your test environment:

```ruby
# Configure static asset server for tests with Cache-Control for performance
config.public_file_server.enabled = true
config.public_file_server.headers = {
  'Cache-Control' => 'public, max-age=3600'
}
```

### config/initializers/wrap_parameters.rb

Add this file with the following contents, if you wish to wrap parameters into a nested hash. This is on by default in new applications.

```ruby
# Be sure to restart your server when you modify this file.
# This file contains settings for ActionController::ParamsWrapper which
# is enabled by default.

# Enable parameter wrapping for JSON. You can disable this by setting :format to an empty array.
ActiveSupport.on_load(:action_controller) do
  wrap_parameters format: [:json]
end

# Disable root element in JSON by default.
ActiveSupport.on_load(:active_record) do
  self.include_root_in_json = false
end
```

### config/initializers/session_store.rb

You need to change your session key to something new, or remove all sessions:

```ruby
# in config/initializers/session_store.rb
AppName::Application.config.session_store :cookie_store, key: 'SOMETHINGNEW'
```

or

```bash
$ bin/rake db:sessions:clear
```

### Remove :cache and :concat options in asset helpers references in views

* With the Asset Pipeline the :cache and :concat options aren't used anymore, delete these options from your views.
