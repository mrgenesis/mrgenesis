---
layout: post
title:  WordPress em Vários Idiomas
categories: [WordPress]
excerpt: Neste post me proponho a apresentar para você uma maneira simples de criar um sistema multilíngua em seu WordPress. Será fácil de criar e de gerenciar desde que você mantenha apenas dois ou três idiomas.

---
Este post é para mostar como fazer um site/blog em WordPress em vários idiomas. Não usaremos nenhum plugin, somente `PHP` e `JavaScript`. Neste artigo eu falo sobre custom taxonomy, meta box, e muito mais. Se você pretender fazer um blog com mais de 3 idiomas, talvez esta não seja a maneira mais indicada, mas vale a pena conferir a solução mostrada neste post.  
A abordagem usada aqui é para ser implementada num tema, ou tema filho, mas nada impede que use este mesmo código num plugin. Sendo assim, ao fim deste artigo você terá criado o seguinte em seu tema:

#### Arquivos:

1.  `DefineIdioma.php`  
    A classe para definir as variáveis  `IDIOMA_PADRAO`  e  `IDIOMA`
2.  `idioma.js`  
    O código JavaScript para gerar um link que direciona o usuário entre os contúdos de idiomas diferente
3.  `archive-mudaridioma.php`  
    O código que processa as informações geradas pelo JavaScript e pesquisa no Banco de Dados qual o conteúdo relacionado no idioma solicitado

#### Recursos:

1.  Nova categoria  `idiomas`  
    Esta cotegoria vai ficar disponível em  `post`  e  `page`
2.  Novo campo  
    Este novo campo é um meta box para armazenas a  `id`  do conteúdo relacionado que ficará disponível em  `post`  e  `page`
3.  Novo  `post_type`  
    Criação da rota  `/mudaridioma/`  que será direcionada para  `archive-mudaridioma.php`

----------

**IMPORTANTE!**  
_O cógido dos artigos foram criados pensando em um objetivo: explicar. Não use “copy/paste” porque isso não é legal; é necessário  **enteder**. Se usar “copy/paste”, 1) você pode quebrar seu site 2) seu código terá algumas coisas erradas como, por exemplo, uso de barras (`/`) ao invés de variáveis (`DIRECTOR_SEPARATOR`), que tornam o código partável. Então,  **entenda**  este artigo e faça sua implementação de forma customzada._

----------

Pronto para escrever muito código? Então, mãos na massa!
## Definindo idiomas
A primeira coisa a fazer é definir um idioma padrão. Na prática, este idioma dará um auxílio na definição do idioma a ser exibido para o usuário final.  
Crie um arquivo novo para criar nossa classe que vai manipular o idioma padrao bem como o idioma a ser exibido aos usuários.  
O nome do arquivo deve ser o mesmo nome da classe. Neste caso seria `DefineIdioma.php`.
```php
<?php // ./class/DefineIdioma.php
class DefineIdioma  
{  
 private $idiomaPadrao = 'pt-BR';  
 private $idiomaUsuario;  
  
 function __construct(){  
 define('IDIOMA_PADRAO', $this->idiomaPadrao);  
 }  
  
 public function defineIdiomaUsuario($idiomaUsuario, $postId)  
 {  
 if (isset($idiomaUsuario)) {  
 $this->idiomaUsuario = $idiomaUsuario;  
 } else {  
 $taxIdioma = wp_get_post_terms($postId, 'idiomas', array('fields' => 'all'));  
 $idioma = $taxIdioma[0]->name;  
 if ($idioma == '') {  
 $this->idiomaUsuario = $this->idiomaPadrao;  
 } else {  
 $this->idiomaUsuario = $idioma;  
 }  
 }  
 define('IDIOMA', $this->idiomaUsuario);  
 }  
 public function getIdiomaPadrao()  
 {  
 return $this->idiomaPadrao;  
 }  
 public function getIdiomaUsuario()  
 {  
 return $this->idiomaUsuario;  
 }  
}
```

O código a cima criará um objeto para definir as variáveis que manipularão o idioma padrão do blog, bem como o idioma a ser exibido para o usuário. O  `pt-BR`  será o padrão meramente porque eu quero assim. Você poderá escolher outro se quiser.

### Idioma Padrão

É muito importante configurarmos o idioma padrão porque, no futuro, quando nenhum idioma for definido, nós o usaremos como base para gerarmos a página corretamente. Por que isto é importante? Pense no seguinte: num site que é somente em português, e passará a ser tanto em português como em Inglês, você  **_não terá as coisas configuradas nos artigos em português_** , de forma que se você não tiver algo padronizado, terá que editar todos os conteúdos em português para se adequar ao novo recurso. Minha proposta é fazer de um jeito que você não tenha que mexer em nada do que já está funcionando.

Pois bem, observe na linha 7 que o simples fato de instanciarmos a classe, ela já definirá nosso idioma padrão. Com isso, nós teremos este recurso nas variáveis  `IDIOMA_PADRAO`  e  `$nome_variavel->getIdiomaPadrao()`. Para configurar o idioma padrão, basta instanciar a classe.

```php
<?php //  ./functions.php
  
include_once(get_template_directory() . '/class/DefineIdioma.php');  
  
add_action('init', function(){  
 global $idioma;  
 $idioma = new DefineIdioma();  
}); 
``` 
Note que criei uma variável global para usarmos ela de qualquer lugar e sempre que for necessário. Já temos o idioma padrão disponível globalmente das seguintes formas:
```php
<?php  //     exemplos.php
echo IDIOMA_PADRAO;  
  
// ou também  
global $idioma;  
echo $idioma->getIdiomaPadrao();  
```
Por padrão, uma  [constante é global](https://www.php.net/manual/pt_BR/language.constants.php), então você pode acessá-la de todo lugar. Então tanto faz se você acessar estes valores pela variário global ou pela constante.

### Idioma do usuário

Para o idioma a ser exibido para o usuário, deve-se chamar o método  `defineIdiomaUsuario()`  na linha 11.  
Note que é muito importante não deixar escapar nada, se não você poderá ter problemas. Você não pode, em hipóse alguma, deixar parte do blog num idioma e parte em outro. Sendo assim, precisamos validar algumas coisas visando definir tudo certinho.  
Observe que no método  `defineIdiomaUsuario()`  (linha 11), há dois maneiras de definir o idioma:

1.  explicitamente através de um parâmetro passado como valor (linha 14):  
    Esta é uma maneira excelente, porque evita fazer qualquer consulta, bem como validações, tornando o código mais performático
2.  através do idioma aplicado no conteúdo (linha 17):  
    Observe que realizamos uma busca na categoria  `'idiomas'`, na primeira posição  `[0]`, para saber qual é o idioma aplicado no conteúdo. Se tiver em braco, aplicamos o padrão do blog.

Nota: neste ponto é que entendemos que para os conteúdos  **_desconfigurados_**, aplicaremos o idioma padrão. Leia o tópico anterior novamemente (Idioma Padrão) e compare com a linha 18.

Continue lendo que isto vai ficar ainda mais claro!

Uma outra coisa que deve ser reforçado é que sempre que possível o idioma deve ser definido explicitamente para que tudo seja resolvido na linha 14.  **Isto tornará o processo performático**.

----------

**IMPORTANTE!**

-   _Por que devo usar o resultado somente da posição [0]_? R: Poque não existe a possibilidade de definir mais de um idioma para o conteúdo. Se isso for feito será um erro do editor do artigo.
    
-   _Por que só criei o idioma padrão até agora, e não criei ainda o idioma do usuário?_  
    R: Porque preciso passar  **a  `id`  do post**  como parâmetro, e esta informação ainda  **não estará disponível**  no arquivo  `functions.php`.
    

----------

Poderíamos usar o arquivo  `header.php`  que é processado antes de todos os outros. Este será o melhor lugar para configurar a variável que determina o idioma do usuário.  
Sendo assim, usaremos o gancho  `head`para evitar editar o  `header.php`  diretamente.
```php
<?php  //      ./functions.php
add_action('wp_head', function(){    
	 // quando este método for executado no gancho "head", você terá a id disponível  
	 $idioma->defineIdiomaUsuario($_GET['lang'], $post->ID);
});  
```
Agora teremos os dois valores disponíveis em toda parte através das constantes  `IDIOMA_PADRAO`  e  `IDIOMA`, ou  `$idioma->getIdiomaPadrao()`  e  `$idioma->getIdiomaUsuario()`.

## Crie uma nova categoria

Ao adicionar uma nova  `taxonomia`, você poderá criar  `queries`  para validar os dados mais facilmente. Então vamos configurá-la.
```php
<?php  // ./functions.php
register_taxonomy('idiomas',  
	array('page', 'post'),  
		array(  
			'hierarchical' => true,  
			'label' => 'Idiomas',  
			'singular_label' => 'Idioma',  
			'rewrite' => false  
		 )  
 );  
```
![](https://s3-sa-east-1.amazonaws.com/mrgenesis.net/wordpress/WordPress-em-varios-idiomas/campo-adicional-para-post-page.png "Categoria idiomas - Novo campo em post e page")
O código a cima gera um campo adicional na tela de edição de posts e páginas, onde pode ser definido o idioma do conteúdo.  **Se este campo não for definido pelo editor, o idioma que será exibido será  `IDIOMA_PADRAO`**  que estaria em  `./class/DefineIdioma.php`, na linha 4.

E se for definido mais de 1 idioma (eu não acredito que alguém poderia fazer uma barbaridade dessas), o idioma que será definido será o que estiver na posição zero  `[0]`, que estaria definido em  `./class/DefineIdioma.php`, na linha 17.

## Configurando as pastas
É importante ressaltar que a constante é global, então estará disponível em toda parte. Sendo assim, você pode usá-la em vários arquivos diferentes. Assim, para definir o  `path`  correspondente ao idioma, bem como quaisquer outros recursos, você pode usar a constante facilmente.  
Note que  `IDIOMA`  é a mesma coisa que  `$idioma->getIdiomaUsuario()`.
```php
<?php //    exemplo.php
$path = '/parts/' . IDIOMA .  '/';  
define('PASTA', $path);  
```
  
Isto atribuiria à variável `PASTA` o seguinte valor: `'/parts/pt-BR/'`.

visualize esta estrutura de pastas abaixo:
```bash
seu-tema/  
├─┬ parts/  
│ └─┬ pt-BR/  
│   ├ en-US/  
├── exemplo.php  
```
Analise o código a cima (`exemplo.php`) onde é definido a constante  `PASTA`, e tente imageinar que você queira exibir em sua homepage um banner com uma imagem bem bonita, e um texto do tipo “Conheça nossos serviços”. Como estamos falando de um site em mais de um idioma, você vai precisar escrever este texto nos idiomas disponíveis.
```php
<?php
// ./parts/pt-BR/banner-servicos.php
echo '<p>Conheça nossos serviços</p>';  

// ./parts/en-US/banner-servicos.php
echo '<p>Learn about our services</p>';  
```
Sempre que precisarmos recuperar arquivos em determinado idioma usaremos a variável  `PASTA`  para isso. Veja:
```php
<?php  //   exemplo.php
get_template_part(PASTA . 'banner-servicos');  
// O resultado será '/parts/pt-BR/banner-servicos.php' ou '/parts/en-US/banner-servicos.php'  
```
Repare a estrutura das pastas.

## Configurando o menu

Com este princípio em mente, devemos cuidar das definições do menu do blog.  
Poderíamos criar os menus de forma dinâmica da seguinte forma:
```php
<?php  //    ./functions.php
register_nav_menus(  
	array(  
		'header-pt-BR' => 'Menu do Cabeçalho em Português',  
		'header-pt-BR' => 'Menu do Roda pá em Português'  
		'footer-en-US' => 'Menu do Cabeçalho em Inglês'  
		'footer-en-US' => 'Menu do Roda pé em Inglês'  
	)  
);  
```
Note que eu usei as palavras  `pt-BR`  e  `en-US`  para identificar o menu. Fiz isso de propósito, porque na hora de usar este menu fica simples. Poderíamos fazer assim:

este código deve ser colocado onde o menu será exibido
```php
<?php  
wp_nav_menu(  
 array(  
 'theme_location' => 'header-' . IDIOMA,  
 'container'      => 'div',  
 'container_class'=> 'sua-class-css',  
 'items_wrap'     => '<ul class="sua-class-css">%3$s</ul>'  
 )  
);  
```
Agora basta configurar as páginas dentro do painel do WordPress em  **Aparência > menu**.

Mamão com açucar!

## Crie um novo campo personalizado

Vamos criar um novo campo para  `post`  e  `page`. Este campo vai armazenar uma chave com nome de  `item-relacionado`. O valor armazenado nesta chave será a  `id`  do conteúdo escrito no idioma primário, ou seja, o blog tem um idioma padrão (`IDIOMA_PADRAO`), e sempre haverá um conteúdo primário escrito no idioma padrão. Em outras palavras, no meu caso, o idioma primário é o  `pt-BR`. Vamos adicionar o novo campo da seguinte forma:
```php
<?php   //    ./functions.php
  
add_action( 'add_meta_boxes', 'addMetaBox' );  
add_action( 'save_post', 'salvaDado' );  
  
/* Adiciona uma meta box na coluna principal das telas de post e page */  
function  addMetaBox()   
{  
 $screens = array( 'posts-pt-br', 'post', 'page' );  
 foreach ($screens as $screen) {  
 add_meta_box(  
 'item-relacionado',  
 'ID do Item relacionados',  
 'exibeFormItemRelacionado',  
 $screen  
 );  
 }  
}  
  
/* Imprime o conteúdo da meta box */  
function  exibeFormItemRelacionado( $post )   
{  
 // Faz a verificação através do nonce  
 wp_nonce_field( plugin_basename( __FILE__ ), 'meu_noncename' );  
 // Os campos para inserção dos dados  
 // Use get_post_meta para para recuperar um valor existente no banco de dados e usá-lo dentro do atributo HTML 'value'  
 $value = get_post_meta( $post->ID, 'item-relacionado', true );  
 echo  '<label for="item-relacionado">';  
 _e('Adicione um número: ' );  
 echo  '</label> ';  
 echo  '<input type="number" id="item-relacionado" name="item-relacionado" value="'.esc_attr($value).'" size="10" />';  
}  
/* Quando o post for salvo, salvamos também nossos dados personalizados */  
function  salvaDado( $post_id )   
{  
 // É necessário verificar se o usuário está autorizado a fazer isso  
 if ( 'page' == $_POST['post_type'] ) {  
 if ( ! current_user_can( 'edit_page', $post_id ) )  
 return;  
 } else {  
 if ( ! current_user_can( 'edit_post', $post_id ) )  
 return;  
 }  
 // Agora, precisamos verificar se o usuário realmente quer trocar esse valor  
 if ( ! isset( $_POST['meu_noncename'] ) || ! wp_verify_nonce( $_POST['meu_noncename'], plugin_basename( __FILE__ ) ) )  
 return;  
 // Por fim, salvamos o valor no banco  
 // Recebe o ID do post  
 $post_ID = $_POST['post_ID'];  
 // Remove caracteres indesejados  
 $mydata = sanitize_text_field( $_POST['item-relacionado'] );  
  
 // Adicionamos ou atualizados o $mydata  
 update_post_meta($post_ID, 'item-relacionado', $mydata);  
}  
```
Veja a explicação do código a cima na  [documentação](https://codex.wordpress.org/pt-br:add_meta_box)  (confesso que fiz um copy/paste). Tem até um exemplo usando classe. Agora temos um novo campo sendo exibido em  `post`  e  `page`, com a seguinte informação: “**ID do item relacionado:**  “.

![]([https://s3-sa-east-1.amazonaws.com/mrgenesis.net/wordpress/WordPress-em-varios-idiomas/meta-box.png](https://s3-sa-east-1.amazonaws.com/mrgenesis.net/wordpress/WordPress-em-varios-idiomas/meta-box.png))

Este campos deve ser preenchido com a  `id`  do conteúdo primário. Por exemplo, no post “Olá Mundo”, que é o conteúdo primário, deve ser deixado em branco, mas em “Hello World” deve ser preenchido a  `id`  do post “Olá Mundo”. Então, digamos que a  `id`  de “Olá Mundo” seja 10. Em “Hello World” deve haver uma chave  `item-relacionado = 10`  gravada no banco de dados.

## Crie a página para fazer o redirecionamento

Observe que você ainda só configurou as definições de idioma. Quando você quiser fazer um redirecionamento entre idiomas, vai precisar de uma página específicamente para isto. Então poderá criar esta página e escondê-la do menu, ou seja, é uma página que existe somente paga esta finalidade. É nela que serão colocadas as  **regras de direcionamentos**. Em seu  `functions.php`  você deve colocar o seguinte código:
```php
<?php  //    ./functions.php
add_action('init', function(){  
 $labels = array( 'name' => 'mudaridioma');  
 $args = array(  
 'labels' => $labels,  
 'public' => true,  
 'public_query' => true,  
 'show_ui' => false,//esconde do nemu  
 'query_var' => true,  
 'rewrite' => true,  
 'capability' => 'post',  
 'has_archive' => true //permite usar um arquivo (archive-mudaridioma.php)  
 );  
 register_post_type('mudaridioma', $args);  
 flush_rewrite_rules();  
 $urlDirecionamento = site_url() .  DIRECTORY_SEPARATOR . 'mudaridioma';  
 define('URL_DIRECIONAMENTOS', $urlDirecionamento);  
});  
```
Com o código a cima você tem uma página para direcionamentos entre idioma. Ainda definiu uma constante para facilitar sua vida no futuro. A constante  `URL_DIRECIONAMENTOS`  tem o seguinte valor:  `https://seu-dominio/mudaridioma`. Crie um arquivo na raiz do tema com o nome  `archive-mudaridioma.php`  para processar as requisições deste link  `https://seu-dominio/mudaridioma`.

## Adicione as regras do redirecionamento entre idiomas

Imagine o seguinte: eu quero que o seguinte post  `https://seu-dominio/ola-mundo/`  direcione para  `https://seu-dominio/hello-world/`. Para fazer isso devemos definir alguns parâmetros obrigatórios. São eles:

1.  O idioma  **atual**  deve,  **obrigatóriamente**, ser definido
2.  O idioma  **pretendido**  também deve,  **obrigatóriamente**, ser definido (vamos chamar de  `lang`)
3.  A  `id`  do post atual
4.  O link atual

Com estas regras atendidas, fica fácil fazer os direcionamentos correto. Você só precisaria gerar um link da seguinte forma:  
**`https://seu-dominio/mudaridioma/`**`?idiomaAtual=pt-BR&lang=en-US&idItemAtual=1&urlAtual=https://site/ola-mundo/`.

Quando fizermos o código em  `javasript`, você vai entender como vamos pegar estes valores dinamicamente.  
A idéia aqui é direcionar este usuário para a postagem correspondente. Vejamos como ficaria o código.  
**Coloque o código a baixo em  `archive-mudaridioma.php`**.
```php
<?php  //    ./archive-mudaridioma.php
  
$idItemAtual = $_GET['idItemAtual'];  
$idiomaAtual = $_GET['idiomaAtual'];  
$urlAtual    = $_GET['urlAtual'];  
$lang = $_GET['lang']; // Idioma Solicitado   
  
// pega o tipo do post  
$tipoPost = get_post_type($idItemAtual);  
  
  
if ($idiomaAtual == IDIOMA_PADRAO) {  
 $idItemPrimario = $idItemAtual;   
} else {  
 $idItemPrimario = get_post_meta($idItemAtual, 'item-relacionado', true);  
}  
  
if ($lang == IDIOMA_PADRAO) {  
    
 $linkPretendido = get_permalink($idItemPrimario);  
    
} else {  
  
 // retorna um array de objetos com todos os posts  
 // que atendem aos critérios definidos a baixo  
 $args = array(  
 'post_per_page' => -1,// todos os posts  
 'meta_key' => 'item-relacionado', // a chave personalizada  
 'meta_value' => $idItemPrimario, // id do post principal  
 'post_type' => $tipoPost, // pode ser page ou post  
 'post_status' => 'publish' // deve estar publicado  
 );  
  
 $objetos = get_posts($args);  
 foreach ($objetos as $objeto) {  
 $idiomaPost = wp_get_post_terms($objeto->ID, 'idiomas', array('fields' => 'all'));  
    
 if ($idiomaPost[0]->name == $lang) {  
  
 $linkPretendido = get_permalink($objeto->ID);  
 break;  
  
 }  
 }  
}  
if (!isset($linkPretendido)) {  
 $linkPretendido  .=  '?lang=' . $lang;  
 echo  '<script>window.location.href = "'.$linkPretendido.'";</script>';  
} else {  
  
 // se eu não achar um item relacionado  
 // redireciono o usuário para o mesmo link de onde ele veio  
 echo  '<script>window.location.href = "' . $urlAtual . '?lang=' . $idiomaAtual . '";</script>';  
}  
```
Lembra que comentamos que definir o idioma padrão é importante? Então… são nessas horas que percebemos isto.  **A lógica do nosso código consiste em relacionar os conteúdos dos outros idiomas ao nosso conteúdo escrito no idioma padrão**. Este relacionamento se dá por meio da  `id`. Por isso que precisamos da  `id`  primária, que se refere a  `id`  do conteúdo primário.

Observe na linha 12: se o idioma do conteúdo atual é o padrão. Então ele mesmo tem a  `id`  primária, que seria definido na linha 13. Assim, minha primeira busca é pela  `id`  primária.  
Caso a condição da linha 12 seja  `false`, você vai buscar a  `id`  do conteúdo primária no post/page em si mesmo (linha 15).

Com a  `id`  primária definida, posso verificar se estou querendo ir para o idioma padrão ou não (linha 18). Caso posistivo, na linha 20, encontramos o link que procurávamos, e então podemos pular para a linha 47, e agilizar o próximo processamento por definir explicitamente o idioma do usuário, em seguida, na linha 48, direcionamos o usuário à página correta.

E se o conteúdo pretendido não for o conteúdo do idioma padrão? Neste caso presisaríamos buscar todos os posts que possuem o item relacionado igual a  `id`  primária. Estes posts serão armazenados na variável  `$objetos`, linha 34. Então você vai precisar fazer uma busca, e achar qual conteúdo tem o idioma definido na categoria do tipo  `idiomas`  que seja igual ao idioma pretendido (`$lang`), linha 38. Quando eu encontrar, gero o link para o redirecionamento, e passo para a linha 48…

Mas… vamos supor que não haja nenhum conteúdo naquele idioma pretendido. Aliás, isso não é nada impossível, partindo do princípio que o blog era em português e agora passou a ser também em inglês. Se nós pensarmos que nem todos os conteúdos foram traduzidos ainda, poderíamos, facilmente, entender que a variável  `$linkPretendido`  só será definida se nós encontrarmos o conteúdo prucurado. Então, para não tornar este post ainda mais longo, caso esta variável não seja definida, que vai significar que o conteúdo procurado não existe, nós só vamos direcionar o usuário para o link de onde ele veio, linha 53. Mas você é fortemente aconselhado a fazer um tratativa mais inteligente.

## Adicione informações relevantes no HTML

Estamos quase lá!

Você precisa adicionar informações no HTML para serem processadas pelo  `javascript`. Uma maneira fácil de fazer isso é por criar um  `input`  escondido.

deve ser adicionado em todas as páginas

```php 
<input type="hidden" data-url="<?php echo URL_DIRECIONAMENTO ?>" data-idioma-atual="<?php echo IDIOMA; ?>" data-id-item-atual="<?php echo get_the_ID(); ?>">  
```
Nos botões você pode adicionar os dados específicos de cada idioma. Vou criar só dois botões, mas você poderá criar quantos quiser, de acordo com os idiomas do teu projeto.

deve ser adicionado em todas as páginas
```php
<!-- Botão pt-BR -->  
<a href="javascript:void(0)" onclick="<?php echo (IDIOMA != 'pt-BR') ? 'redireciona().pronto(this)': ''; ?>" data-idioma-pretendido="pt-BR"><imag src="/pt-BR.ico" /></a>  
  
<!-- Botão en-US -->  
<a href="javascript:void(0)" onclick="<?php echo (IDIOMA != 'en-US') ? 'redireciona().pronto(this)': ''; ?>" data-idioma-pretendido="en-US"><imag src="/en-US.ico" /></a>  
```
Temos tudo pronto para criarmos o código em  `javascript`.

## Gere o link para troca de idioma

Primeiro adicione o arquivo  `javascript`  no  `header.php`.
```php
<?php //     ./functions.php
add_action('wp_head', function(){  
 echo '<script  src="' . get_template_directory() . '/js/idioma.js"></script>';  
});  
```
Agora abra o arquivo e escreva o código para pegar as informações no  `HTML`.
```js 
//    ./js/idioma.js
function  redireciona(){  
 let  dados  =  document.querySelector("#dados")  
 return {   
 url:  function(dados){  
 return  dados.getAttribute("data-url")  
 },  
 urlAtual:  function(){  
 return  document.URL  
 },  
 idiomaAtual:  function(dados){  
 return  dados.getAttribute("data-idioma-atual")  
 },  
 lang:  function(elemento){ // Idioma solicitado  
 return  elemento.getAttribute("data-idioma-solicitado")  
 },  
 idItemAtual:  function(dados){  
 return  dados.getAttribute("data-id-item-atual")  
 },  
 pronto:  function(elemento){  
 let  link;  
 link  =  this.url(dados)  
 link  +=  "?urlAtual="  +  this.urlAtual()  
 link  +=  "&idiomaAtual="  +  this.idiomaAtual(dados)  
 link  +=  "&lang="  +  this.lang(elemento) // Idioma solicitado  
 link  +=  "&idItemAtual="  +  this.idItemAtual(dados)  
 window.location.href  =  link  
 }  
 }  
}  
```
No fim das contas, a variável  `link`  terá um valor semelhante a este:  
`https://seu-dominio/mudaridioma/?idiomaAtual=pt-BR&lang=en-US&idItemAtual=1&urlAtual=https://site/ola-mundo/`.

Estas informações serão processadas em  `archive-mudaridioma.php`.

Comente o que você achou do post
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTM3Njc3MjUsOTczMjMwMDddfQ==
-->