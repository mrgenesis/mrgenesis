---
layout: post
title:  WordPress em Vários Idiomas
categories: [WordPress]
excerpt: Neste post me proponho a apresentar para você uma maneira simples de criar um sistema multilíngua em seu WordPress. Será fácil de criar e de gerenciar desde que você mantenha apenas dois ou três idiomas.
---
Este post é para mostrar como fazer um site/blog em WordPress em vários idiomas. Não usaremos nenhum plugin, somente `PHP` e `JavaScript`.   
Se você pretender fazer um blog com mais de 3 idiomas, este post não servirá para você. Te aconselho usar outro método.  

## Definindo idiomas
A primeira coisa a fazer é definir um idioma padrão. Na prática, este idioma dará um auxílio na definição do idioma a ser exibido para o usuário final.
Crie um arquivo novo para criar nossa class que definirá o idioma padrão bem como o idioma a ser exibido ao usuário.
```php
<?php
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
O código a cima criará um objeto para definir as variáveis que manipularão o idioma padrão do blog, bem como o idioma a ser exibido para o usuário. O `pt-BR` será o padrão meramente porque eu quero assim. Você poderá escolher outro se quiser.
Para configurar o idioma padrão, basta instanciar a classe, e para o idioma a ser exibido para o usuário, deve-se chamar o método `defineIdiomaUsuario()`.
Note que é muito importante não deixar escapar nada, se não você poderá ter problemas. Você não pode, em hipóse alguma, deixar parte do blog num idioma e parte em outro. Sendo assim, precisamos validar algumas coisas visando definir tudo certinho. 
Observe que no método `defineIdiomaUsuario()`, há dois maneiras de definir o idioma:
1. explicitamente através da variável de um parâmetro:
Esta é uma maneira excelente, porque evita fazer qualquer consulta, bem como validações
2. através do idioma aplicado no conteúdo:
Observe que realizamos uma busca na categoria `'idiomas'`, na primeira posição `[0]`, para saber qual é o idioma aplicado no conteúdo. Se tiver em braco, aplicamos o padrão do blog.

Então, antes de fazer qualquer coisa preciso instaciar a classe da seguinte forma:
``` php
<?php
// escreva este código em functions.php

include_once(get_template_directory() .  DIRECTORY_SEPARATOR  . 'DefineIdioma.php');

add_action('init', function(){
	global $idioma;
	$idioma = new DefineIdioma();
});
 ```
Note que criei uma variável global para usarmos ela de qualquer lugar e sempre que for necessário. Já temos o idioma padrão disponível globalmente da seguinte forma:
```php
global $idioma;
$idioma->getIdiomaPadrao();
```

Assim para definir o idioma do usuário seria da seguinte maneira:
```php
global $idioma;
$idioma->defineIdiomaUsuario($_GET['lang', $post->ID);
```

Vale reforçar que se o idioma estiver sendo explicitamente solicitado, eu posso evitar fazer `queries` no banco, tornando meu código mais performático. Mas se não houver uma definição explicitada eu busco o idioma que foi definido na `taxinomia` com nome `idiomas` (falaremos sobre isso depois). E se o idioma não foi definido dentro do post, o padrão será mostrado. 

Se você quiser resumir um pouco para escrever menos, poderá acessar estes valores através das constantes `IDIOMA_PADRAO` e `IDIOMA`.
Por padrão, uma [constante é global](https://www.php.net/manual/pt_BR/language.constants.php), então você pode acessá-la de todo lugar.

Agora que temos estes dois valores configurados globalmente, podemos configurar outras coisas.
**Lembre-se de chamar o método para definir o idioma no arquivo `header.php`, antes de qualquer outra coisa**. Se você gerar suas páginas sem ter definido o idioma do usuário, seu blog vai quebrar.
## Crie uma nova categoria
Ao adicionar uma nova `taxonomia`, você poderá criar `queries` para validar os dados mais facilmente. Então vamos configurá-la 
```php
<?php
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
O código a cima gera um campo adicional na tela de edição de posts e páginas, onde pode ser definido o idioma do conteúdo. **Se este campo não for definido pelo editor, o idioma que será exibido será `$idiomaPadrao`** que está na classe `DefineIdioma`.

## Configurando as pastas
 É importante ressaltar que, como o objeto foi definido como global, ele estará disponível em toda parte. Sendo assim, você pode usá-lo em vários arquivos diferentes. Assim para definir o `path` correspondente ao idioma, bem como quaisquer outros recursos, você pode usar o objeto facilmente.
`IDIOMA` é a mesma coisa que `$idioma->getIdiomaUsuario()`.
 ``` php
 $path = get_template_directory() . DIRECTORY_SEPARATOR . IDIOMA .  DIRECTORY_SEPARATOR;
 define('PASTA', $path);
```
Isto atribuiria à variável `PASTA` o seguinte valor: 
`'wp-content/themes/seu-tema/parts/pt-BR/'`.

 ```
seu-tema/
├─┬ parts/
│ └─┬ pt-BR/
│   ├ en-US/
├── header.php
```
Sempre que precisarmos recuperar arquivos em determinado idioma usaremos a variável `PASTA` para isso. Veja:
```php
get_template_part(PASTA . 'exemplo-header');
```

## Configurando o menu

Com este princípio em mente, devemos cuidar das definições do menu do blog.
Poderíamos criar os menus de forma dinâmica da seguinte forma:
```php
// functions.php
register_nav_menus(
	array(
	'header-pt-BR' => 'Cabeçalho em Português',
	'header-en-US' => 'Cabeçalho em Inglês'
	)
);
```

Note que eu usei as palavras `pt-BR` e `en-US` para identificar o menu. Fiz isso de propósito, porque na hora de usar este menu fica simples. Poderíamos fazer assim:
```php
// este código deve ser colocado onde o menu será exibido 
wp_nav_menu(
	array(
	'theme_location' => 'header-' . IDIOMA,
	'container'      => 'div',
	'container_class'=> 'sua-class-css',
	'items_wrap'     => '<ul class="sua-class-css">%3$s</ul>'
	)
);
```
Agora basta configurar as páginas dentro do painel do WordPress em **Aparência > menu**.
Mamão com açucar!
## Crie um novo campo personalizado
Vamos criar um novo campo para `post` e `page`. Este campo vai armazenar uma chave com nome de `item-relacionado`. O valor armazenado nesta chave será o `id` do conteúdo escrito no idioma primário, ou seja, o blog tem um idioma padrão (`IDIOMA_PADRAO`), e sempre haverá um conteúdo primário escrito no idioma padrão. Em outras palavras, no meu caso, o idioma primário é o `pt-BR`.
Vamos adicionar o novo campo da seguinte forma:
```php
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
Veja a explicação do código a cima na [documentação](https://codex.wordpress.org/pt-br:add_meta_box).
Agora temos um novo campo sendo exibido em `post` e `page`, com a seguinte informação: **Adicione um número:** ".
Este campos deve ser preenchido com a `id` do conteúdo primário. Por exemplo, no post "Olá Mundo", que é o conteúdo primário, deve ser deixado em branco, mas em "Hello World" deve ser preenchido a `id` do post "Olá Mundo". Então, digamos que a `id` de "Olá Mundo" seja 10. Em "Hello World" deve haver uma chave `item-relacionado = 10` gravada no banco de dados.
## Crie a página para fazer o redirecionamento
Observe que você ainda só configurou as definições de idioma. Quando você quiser fazer um redirecionamento entre idiomas, vai precisar de uma página específicamente para isto. Então poderá criar está página e escondê-la do menu, ou seja, é uma página que exeste somente paga esta finalidade. É nela que será colocada as **regras de redirecionamentos**.
Em seu `functions.php`você deve colocar o seguinte código:
```php
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
		'has_archive' => true //permite usar um arquivo
	);
	register_post_type('mudaridioma', $args);
	flush_rewrite_rules();
	$urlDirecionamento = site_url() .  DIRECTORY_SEPARATOR . 'mudaridioma';
	define('URL_DIRECIONAMENTOS', $urlDirecionamento);
});
```
Com o código a cima você tem uma página para redirecionamento entre idioma. Ainda definou um constante para facilitar sua vida no futuro. A constante `URL_DIRECIONAMENTOS` tem o seguinte valor: `https://seu-dominio/mudaridioma`. Crie um arquivo na rai do tema com o nome `archive-mudaridioma.php` para processar as requisições deste link  `https://seu-dominio/mudaridioma`.

## Adicione as regras do redirecionamento entre idiomas
Imagine o seguinte: eu quero que o seguinte post `https://site.com/ola-mundo/` direcione para `https://site.com/hello-world/`.
Para fazer isso devemos definir alguns parâmetros obrigatórios. São eles:

1. O idioma **atual** deve, **obrigatóriamente**, ser definido
2. O idioma **pretendido** também deve, **obrigatóriamente**, ser definido
3. A `id` do post atual

Com estas regras atendidas, fica fácil fazer os redirecionamentos corretamente. Você só precisaria gerar um link da seguinte forma:
`https://site/ola-mundo/?idiomaAtual=pt-BR&lang=en-US&idItemAtual=1&urlAtual=https://site/ola-mundo/`.
``` bash
# veja separadamente
idiomaAtual = pt-BR
lang = en-US
idItemAtual = 1
urlAtual = https://site/ola-mundo/
```
Quando fizermos o código em `javasript`, você vai entender como vamos pegar estes valores dinamicamente.
A idéia aqui é redirecionar este usuário para a postagem correspondente. Vejamos como ficaria o código.
**Coloque o código a baixo em `archive-mudaridioma.php`**.
``` php  

$idItemAtual = $_GET['idItemAtual'];
$idiomaAtual = $_GET['idiomaAtual'];
$urlAtual    = $_GET['urlAtual'];
$lang = $_GET['lang']; // Idioma Solicitado  

// pega o tipo do post
$tipoPost = get_post_type($idItemAtual);

$cont = 0; // Para verificar se eu encontrei o item prucurado

if ($idiomaAtual == IDIOMA_PADRAO) {
	$idItemPrimario = $idItemAtual;	
} else {
	$idItemPrimario = get_post_meta($idItemAtual, 'item-relacionado', true);
}

if ($lang == IDIOMA_PADRAO) {

	$linkPretendido = get_permalink($idItemPrimario);
	++$cont; // adiciona +1 se encontrar o item relacionado
	
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
			$idioma->defineIdiomaUsuario($lang, $objeto->ID);
			$linkPretendido = get_permalink($objeto->ID);
			++$cont; // adiciona +1 se encontrar o item relacionado
			break;
		}
	}
}
if ($cont > 0) {
	$linkPretendido  .=  "?lang=".IDIOMA;
	echo  '<script>window.location.href = "'.$linkPretendido.'";</script>';
} else {

	// se eu não achar um item relacionado
	// redireciono o usuário para o mesmo link de onde ele veio
	echo  '<script>window.location.href = "'.$urlAtual.'";</script>';
}

```
## Adicione informações relevantes no HTML
Estamos quase lá!
Você precisa adicionar informações no HTML para ser processado pelo `javascript`. Uma maneira fácil é por criar um `input` escondido.
```php
<input type="hidden" data-url="<?php echo URL_DIRECIONAMENTO ?>" data-idioma-atual="<?php echo IDIOMA; ?>" data-id-item-atual="<?php echo get_the_ID(); ?>">
```
Nos botões você pode adicionar os dados específicos de cada idioma. Vou criar só dois botões, mas você poderá criar quantos quiser, de acordo com os idiomas do teu projeto.
```php
<!-- Botão pt-BR -->
<a href="javascript:void(0)" onclick="<?php echo (IDIOMA != 'pt-BR') ? 'redireciona().pronto(this)': ''; ?>" data-idioma-pretendido="pt-BR"><imag src="pt-BR.ico" /></a>

<!-- Botão en-US -->
<a href="javascript:void(0)" onclick="<?php echo (IDIOMA != 'en-US') ? 'redireciona().pronto(this)': ''; ?>" data-idioma-pretendido="en-US"><imag src="path/img/en-US.ico" /></a>
```
Temos tudo pronto para criarmos o código em `javascript`.
## Gere o link de para troca de idioma com JavaScript

Primeiro adicione o arquivo `javascript` no `header.php`.

```php
<script  src="<?php  bloginfo('template_url'); ?><?php echo DIRECTORY_SEPARATOR; ?>idioma.js"></script>
```
Agora abra o arquivo e escreva o código para pegar as informações no `HTML`.

```javascript
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
			let  valor;
			valor  =  this.url(dados)
			valor  +=  "?urlAtual="  +  this.urlAtual()
			valor  +=  "&idiomaAtual="  +  this.idiomaAtual(dados)
			valor  +=  "&l="  +  this.lang(elemento) // Idioma solicitado
			valor  +=  "&idItemAtual="  +  this.idItemAtual(dados)
			window.location.href  =  valor
		}
	}
}
```
No fim das contas, a variável `valor` terá um valor semelhante a este:
`https://site/ola-mundo/mudaridioma/?idiomaAtual=pt-BR&lang=en-US&idItemAtual=1&urlAtual=https://site/ola-mundo/`.

Estas informações serão processadas em `archive-mudaridioma.php`.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM2MjE2NjQ2NywxNzM2MTcxNjQ5XX0=
-->