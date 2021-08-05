
# FLUTTER - Monórepo

  

#### Versão 1.0

  

# I - Introdução

  

Este é um projeto de exemplo de Flutter utilizando o conceito de monórepo, ou seja, um único repositorio contendo os projetos do microfrontend

  

A arquitetura do projeto esta baseada no Clean Dart com modular, tendo suas funcionalidades quebradas por módulos.

![Estrutura de pastas](https://user-images.githubusercontent.com/2637049/116998888-40229b80-acb5-11eb-9571-4bfe42aa6cb7.png)


  

Para criação de um novo modulo utilize o comando abaixo no CLI

  

	flutter create --template=package module_name

  

feito isto adicione a referencia dele no **pubspec** do **core**

  

	commons_dependencies:
		path: '../commons_dependencies'

  

Para rodar o projeto execute o **flutter pub get** para baixar as dependências e depois **flutter run**

  

# II - Gerenciamento de Estado

  

O pacote de gerencia de estado foi o **rx_notifier** entretanto devido a utilização do padrão **triple** isso pode ser facilmente modificado bastando trocar a extensão da store pela classe referente a gerencia de estado que preferir (como a **MobxStore** por exemplo), [consultar documentação](https://pub.dev/packages/flutter_triple)

  
**DE**

	class HomeStore extends NotifierStore<Failure, EnterpriseVM> {
	}



**PARA**

	class HomeStore extends MobxStore<Failure, EnterpriseVM> {
	}

  

## Triple Pattern

  

O triple é um pacote que propõe um padrão de desenvolvimento para suas gerencias de estado ![Triple](https://miro.medium.com/max/369/0*QcSmCm5f9h2yNRYG.png)

  

Basicamente ele segmenta o estado da sua aplicação em três, estado de **Load**, estado de **sucesso** e estado de **erro**

  

Com isso podemos facilmente criar nossa classe de estado e utilizamos o **setLoad(bool)** para atualizar o estado de load, o **update(dynamic)** para atualizar o estado de sucesso e o **setError(Failure)** para atualizar o estado de erro

  

	class ProductStore extends StreamStore<Exception, ProductList> {
		final ProductRepository repository;
		
		Counter({required this.repository}): super([]);
		
		fetchProduct() async {
			setLoading(true);
			try {
				final list = await repository.fetchAllProducts();
				update(list);
			} on Exception catch (e) {
				setError(e);
			} finally {
				setLoading(false);
			}
		}
	}

  

## ScopedBuilder

Desta forma quando quisermos construir um widget de forma reativa teremos que utilizar o ScopedBuilder que poderemos segmentar por estado e informar o que faremos em cada segmento do estado da aplicação (erro, sucesso ou load)


	Widget build(BuildContext context){
		return Scaffold(
			body: Center(
				child: ScopedBuilder(
					store: counter,
					onState: (context, state) => Text('$state'),
					onError: (context, error) => Text(error.toString()),
					onLoading: (context) => CircularProgressIndicator(),
				);
			),
		);
	}

  

## Observer

  

Se quisermos segmentar a reatividade para executarmos funções em background podemos utilizar o **observer()**

	store.observer(
		onState: (state) => print(state),
		onError: (error) => print(error),
		onLoading: (loading) => print(loading),
	);

  

Para testes do triple consulte o pacote [triple_test](https://pub.dev/packages/triple_test)


Para mais informações consulte [o artigo](https://medium.com/flutterando/triple-pattern-um-novo-padr%C3%A3o-para-ger%C3%AAncia-de-estado-no-flutter-2e693a0f4c3e)

  

## PACOTES

  

Neste projeto foram utilizados alguns pacotes

  

-  [effective_dart](https://pub.dev/packages/effective_dart) - Auxilia a seguir a guideline do [Dart](https://dart.dev/guides/language/effective-dart)

  

-  [flutter_modular](https://pub.dev/packages/flutter_modular) - Gerencia as rotas e as injeções de dependência, além de ter um sistema de dispose inteligente para eliminar classes que não estão sendo mais utilizadas.

  

-  [flutter_triple](https://pub.dev/packages/flutter_triple) - Auxilia no padrão de gerencia de estado

  

-  [dio](https://pub.dev/packages/dio) - HTTP client poderoso

  

-  [dartz](https://pub.dev/packages/dartz) - Auxilia no retorno de tuplas e também traz o conceito de programação funcional para dentro do DART

  

-  [asuka](https://pub.dev/packages/asuka) - Mostra Snackbars, dialogs, ModalSheets em um único provider de forma simples e limpa

  

-  [flutter_svg](https://pub.dev/packages/flutter_svg) - Renderiza arquivos svg dentro do app

  

-  [hive](https://pub.dev/packages/hive) - Banco de dados local, simples, performático e poderoso

  

-  [build_runner](build_runner) - Pacote utilizado por alguns packages para geração de código (neste caso foi utilizado no mockito) para gerar as classes rodar **flutter pub run build_runner build --delete-conflicting-outputs**

  

-  [mockito](https://pub.dev/packages/mockito) - Pacote que auxilia na geração de Mocks para testes

  

-  [Slidy](https://pub.dev/packages/slidy) - CLI para auxiliar na criação de classes do flutter

  

# III - Estrutura do Projeto

  

A estrutura do projeto esta em monorepo e suas funcionalidade(módulos) quebrado por packages.

 - [base_app](./example/base_app)
 - [commons](./example/commons)
 - [commons_dependencies](./example/commons_dependencies)
 - [core](./example/core)
 - [module_home](./example/module_home)
 - [module_login](./example/module_login)
 - [module_splash](./example/module_splash)



![Estrutura](https://user-images.githubusercontent.com/2637049/117327456-fed2ed00-ae68-11eb-9afd-874c67587955.png)



[DOCUMENTO](https://drive.google.com/file/d/1fsTYSfi2rtystFskGd1oX6lTPOJ_bFo1/view?usp=sharing)

## Base APP

  

O base app é o ponto de entrada da minha aplicação, ele irá conversar somente com o **CORE** que irá acessar os modulos.

Ele irá ter a chamado do runApp()

  

	void main() => runApp(
		ModularApp(
		module: AppModule(),
		child: AppWidget(initialRoute: '/splash'),
		),
	);

  

## CORE

  

O core é o coração do nosso app, ele fará a ponte de conversa entre o **base_app** e os **módulos** é nele que estará a configuração das rotas principais e a chamada para o **MaterialApp()**

  

	Widget build(BuildContext context) {

		return MaterialApp(

		title: 'Flutter Slidy',

		theme: ThemeData(primarySwatch: Colors.blue),

		initialRoute: initialRoute,

		builder: asuka.builder,

		).modular();

	}

  

## Modules

  

Os **módulos** são as quebras de funcionalidades do app.
Eles são responsavei pelas features do app, cada modulo deve possuir **uma e somente uma responsabilidade**

**IMPORTANTE:** Os módulos são totalmente desacoplados e independente de outros modulos e nao podem compartilhar informações entre eles, caso isso seja necessário, colocar no **Commons**

  

## Commons

  

O **commons** é o pacote que será responsável por ter toda a funcionalidade que será compartilhado entre todos os módulos, como interceptors, widgets, stores, etc.


  

## Commons Dependencies


Aqui é onde terá todas as dependências do projeto e os módulos pegam suas dependências daqui, desta forma garantimos que todos os módulos utilizam a mesma versão de package
**É a única dependência que o modulo pode possuir**
  

# IV - Telas

  

![App](https://user-images.githubusercontent.com/2637049/117363594-28a10980-ae93-11eb-89fa-9a70344a5af0.gif)

  

![SplashScreen](https://user-images.githubusercontent.com/2637049/116999011-6fd1a380-acb5-11eb-87b6-c77e5d2ee606.png)


  


![Tela de Login](https://user-images.githubusercontent.com/2637049/116999041-7bbd6580-acb5-11eb-98be-9687309249ab.png)


  
![Home Page](https://user-images.githubusercontent.com/2637049/116999108-8b3cae80-acb5-11eb-8292-43935d7cf724.png)


  

![Tela de Detalhe](https://live.staticflickr.com/65535/51145878610_522f4c73af_b.jpg)
