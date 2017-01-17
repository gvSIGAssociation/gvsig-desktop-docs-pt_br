Acesso a objetos no gvSIG
==========================

Funções current
-----------------

São aquelas funções que nos ajudam a acessar de uma forma rápida a partes de gvSIG que estejam em execução ou carregadas. Tais como a Vista que esteja aberta ou a camada selecionada da Vista.

Estas funções se encentram dentro da biblioteca ``gvsig`` e são:

.. py:function:: currentProject()

	Devolve o projeto atual de gvSIG
	
.. py:function:: currentDocument()

	Devolve o documento atual (Vista ou tabela)
	
.. py:function:: currentView()

	Devolve a Vista ativa ou Não. Se não há Vista ativa devolve um erro de RuntimeException
	
.. py:function:: currentLayer()

	Devolve a camada ativa da Vista

Por exemplo, tendo um Projeto com uma Vista aberta e uma camada carregada e selecionada nesta Vista, executando::

	import gvsig

	def main(*args):

		project = gvsig.currentProject()
		print "project: ", type(project)
		print "project.name: ", project.name

		view = gvsig.currentView()
		print "view: ", type(view)
		print "view.name: ", view.name

		layer = gvsig.currentLayer()
		print "layer: ", type(layer)
		print "layer.name: ", layer.name
		
Dando como resultado no console algo similar a::

	Running script acceso.
	project:  <type 'org.gvsig.app.project.DefaultProject'>
	project.name:  Sin título
	view:  <type 'org.gvsig.app.project.documents.view.DefaultViewDocument'>
	view.name:  Sin título
	layer:  <type 'org.gvsig.fmap.mapcontext.layers.vectorial.FLyrVect'>
	layer.name:  points_layer-57ab8d7e99c
	Script acceso terminated.


Projeto
--------

A classe Project ( :javadoc:`DefaultProject <DefaultProject>` ) se encarrega da gestão dos documentos, Vistas, Tablas, Mapas, Gráficos e outros que podem existir a partir  outras extensões como o de Séries de mapas.

Para acessar esta classe podemos usar :py:func:`currentProject()`, que nos devolverá a instância do projeto que se encontra atualmente aberto no gvSIG.

Por exemplo, podemos obter seu nome e a projeção definida no projeto::


	from gvsig import *

	def main(*args):

		project = currentProject()
		name = project.getName()
		prjcode = project.getProjectionCode()
		prj = project.getProjection()

		print "Project Name: ", name
		print "Projection Code: ", prjcode, type(prjcode)
		print "Projection: ", prj, type(prj)

Mostra no console::

	Project Name:  Sin título
	Projection Code:  EPSG:4326 <type 'unicode'>
	Projection:  EPSG:4326 <type 'org.gvsig.crs.Crs'>
	

Neste caso, `getProjection()` é um método implementado na API de gvSIG, e `getProjectionCode()` é um método colocado na  API de gvSIG a partir das bibliotecas de Jython.	

Documento Vista
---------------

O documento Vista ( :javadoc:`DefaultViewDocument <DefaultViewDocument>` ) conterá as camadas de nosso projeto, nela podemos visualizá-las e editá-las.

Para acessar as Vistas criadas em um Projeto, podemos usar duas funções: :py:func:`currentView()` ou :py:func:`currentDocument()` para acessar a Vista ativa, ou  `currentProject().getView("Nombre")` para acessar a una determinada Vista::

    # encoding: utf-8

    from gvsig import *

    def main(*args):

        project = currentProject()
        
        # Acesso a vista com nome "Vista1"
        view1 = currentProject().getView("Vista1")
        

Uma Vista pode ter diferentes camadas ou serviços carregados. Um Projeto pode ter várias vistas. Por exemplo, com o seguinte script listaremos todas as Vistas que existem em nosso projeto::

	from gvsig import *

	def main(*args):

		project = currentProject()
		views = project().getViews()
		for view in views:
			print view

Também podemos realizar outras operações. Por exemplo, podemos criar uma nova Vista no nosso Projeto e modificar a sua projeção::

	from gvsig import *

	def main(*args):

		project = currentProject()
		
		# Criamos nova vista
		view = project.createView("Nova Vista")
		print "Vista nova: ", view.getName()
		print u"Projeção da Vista: ", view.getProjectionCode()

# Nós utilizamos uma função CRS para obter o objeto #correspondente a um código crs

		newcrs = getCRS("EPSG:32630")
		view.setProjection(newcrs)
		
		print u"Projeção da nova Vista: ", view.getProjectionCode()
		
No caso que já exista una Vista com esse nome, se abrirá um índice. Se voltamos a executar o script anterior, o nome da nova vista será: "Nova Vista - 1".

Se estamos com a Vista anteriormente criada aberta no gvSIG, podemos acessar diretamente esta Vista aberta quando executamos nosso script mediante :py:func:`currentView()`. Por exemplo::


	from gvsig import *

	def main(*args):

		view = currentView()
		print "Nome da Vista: ", view.getName()

		
Podemos centrar a vista em um ponto::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		view = currentView()
		encuadre = createEnvelope([10,10],[20,20])
		view.getMapContext().getViewPort().setEnvelope(encuadre)

		view.centerView(createEnvelope([20,20],[50,50]))
	
	
Documento Tabela
----------------
Outro tipo de documentos que podemos ter no nosso projeto são as Tabelas ( :javadoc:`DefaultFeatureStore <DefaultFeatureStore>` ). Estas tabelas podem fazer referência tanto a tabelas adicionadas no gvSIG como a tabela de atributos das camadas ou outras que apareçam no gestor de projetos.

Da mesma forma que os documentos vista, podemos utilizar a função :py:func:`currentTable()` o :py:func:`currentDocument()` ou `project.getTable("Name")`.


Camada
------

Qualquer camada ou serviço adicionado na nossa Vistas será acessível mediante Scripting. 

Por exemplo, uma operação básica, seria a criação de uma **camada vetorial** e adicioná-la em nova Vista::


	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		# Criamos o esquema para a camada
		ft = createFeatureType()
		ft.append("GEOMETRY", "GEOMETRY")

		# Definimos o tipo da geometria.
		# Usamos as constantes POINT e D2 que se encontram
		# dentro de biblioteca gvsig.geom
		ft.get("GEOMETRY").setGeometryType(POINT, D2)

		# Criamos nova camada com o novo esquema
		# A função se encarrega de estabelecer um path temporal
		shp = createShape(ft)

		# Colocamos a camada a nossa nova vista
		# Será criada com o nome de View 001 por padrão
		newview = currentProject().createView()
		newview.addLayer(shp)

Teremos a opção de iterar sobre todas as camadas que se tenha em uma Vista::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):
		view = currentView()
		layers = view.getLayers()

		# Acessando iterando as camadas
		print "\nIterando: "
		for layer in layers:
			print "\tCamada: ", layer.getName(), 
			print " Tipo: ", layer.getTypeVectorLayer().getFullName()

		# Acessar mediante índices
		print u"\nMediante índices: "
		for i in range(0, len(layers)):
			print "\tCamada: ", layers[i].getName(),
			print " Tipo: ", layers[i].getTypeVectorLayer().getFullName()

Se teremos una Vista com três camadas, o resultado por console será similar ao seguinte::

	Iterando: 
		Camada:  tmpshp-57ae45dd1765  Tipo:  Point:2D
		Camada:  tmpshp-57ae45f712b6  Tipo:  MultiCurve:2D
		Camada:  tmpshp-57ae45fe1112  Tipo:  MultiSurface:3DM

	Mediante índices: 
		Camada:  tmpshp-57ae45dd1765  Tipo:  Point:2D
		Camada:  tmpshp-57ae45f712b6  Tipo:  MultiCurve:2D
		Camada:  tmpshp-57ae45fe1112  Tipo:  MultiSurface:3DM
	
Se queremos acessar as camadas já existentes na Vista podemos fazer mediante ``currentView().getLayer("Nome")``, se ela está  selecionada na tabela de conteúdos (TOC) mediante :py:func:`currentLayer()`
	
Outros métodos que podemos usar sobre uma camada adicionada são os de ``.setVisible(True)`` para modificar sua visibilidade na Vista, ou `` layer.setActive(True)`` para modificar sua seleção dentro da Tabela de Conteúdos.

Grupo de entidades: FeatureSet
------------------------------

Para obter as entidades de uma camada ou tabela, podemos fazer mediante ``layer.features()``, a qual faz uma solicitação ao **store** da camada, e nos devolve um featureSet ( :javadoc:`DefaultFeatureSet <DefaultFeatureSet>` ) com o filtro aplicado sobre a camada ou ordem que  atribuímos. Este featureSet nos permite iterar sobre as entidades da camada.

Depois, por exemplo, podemos acessar estas entidades e a seus valores mediante o método ``getValues()`` sobre cada ``feature``, o qual devolve um dicionário que podemos imprimir::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):
		layer = currentLayer()
		features = layer.features()
		print u"Número de entidades: ", features.getSize()

		for feature in features:
			print feature.getValues()
		
Por console o resultado neste caso será::

	Número de entidades:  7
	{u'NAME': u'Feature1', u'ID': 1L, u'GEOMETRY': POINT (1.0 2.0)}
	{u'NAME': u'Feature2', u'ID': 2L, u'GEOMETRY': POINT (5.0 3.0)}
	{u'NAME': u'Feature2', u'ID': 3L, u'GEOMETRY': POINT (3.0 3.0)}
	{u'NAME': u'Feature2', u'ID': 4L, u'GEOMETRY': POINT (2.0 1.0)}
	{u'NAME': u'Feature3', u'ID': 5L, u'GEOMETRY': POINT (2.0 6.0)}
	{u'NAME': u'Feature3', u'ID': 6L, u'GEOMETRY': POINT (6.0 2.0)}
	{u'NAME': u'Feature3', u'ID': 7L, u'GEOMETRY': POINT (2.0 7.0)}


Podemos fazer diferentes filtros que devolveriam diferentes featureSet. O parâmetro ``expresion`` necessita de um String para fazer o filtro, o parâmetro ``sortBy`` o campo sobre o qual se ordenará o featureSet, o parâmetro ``asc`` ordenará em ordem ascendente ou descendente segundo o campo selecionado.

Uns exemplos de filtros e seus resultados::

	features = layer.features(expresion="ID < 4", sortby="NAME", asc=True)		

	{u'NAME': u'Feature1', u'ID': 1L, u'GEOMETRY': POINT (1.0 2.0)}
	{u'NAME': u'Feature2', u'ID': 2L, u'GEOMETRY': POINT (5.0 3.0)}
	{u'NAME': u'Feature2', u'ID': 3L, u'GEOMETRY': POINT (3.0 3.0)}

	features = layer.features(expresion="ID < 4 AND 1 < ID", sortby="NAME", asc=False)
	
	{u'NAME': u'Feature2', u'ID': 2L, u'GEOMETRY': POINT (5.0 3.0)}
	{u'NAME': u'Feature2', u'ID': 3L, u'GEOMETRY': POINT (3.0 3.0)}

    features = layer.features(expresion="ID < 4 AND NAME != 'Feature1'", asc=True)

	{u'NAME': u'Feature2', u'ID': 2L, u'GEOMETRY': POINT (5.0 3.0)}
	{u'NAME': u'Feature2', u'ID': 3L, u'GEOMETRY': POINT (3.0 3.0)}
	
Outras opções mais avançadas usando a API de gvSIG::

	from gvsig import *

	def main(*args):

		layer = currentLayer() #layer_append_features.main()
		features = layer.features()

		print "\n Show features: "
		for f in features:
			print f

		fquery = layer.getDataStore().createFeatureQuery()
		fquery.addAttributeName("ID")
		fquery.addAttributeName("GEOMETRY")

		#FeatureQuery
		print "\nFQuery"
		#fquery.setLimit(3)
		fset = layer.getDataStore().getFeatureSet(fquery)
		for i in fset:
			print i

		#FeatureQueryOrder
		print "\n Fquery order geometry"
		forder = layer.getDataStore().createFeatureQuery().getOrder()
		print forder
		forder.add("GEOMETRY", True)
		fquery.setOrder(forder)

		fsetorder = layer.getDataStore().getFeatureSet(fquery)

		for i in fsetorder:
			print i

			
Entidade
--------

Já vimos anteriormente que podemos acessar os valores de cada entidade ( :javadoc:`DefaultFeature <DefaultFeature>` ) com ``getValues()``, mas também podemos acessar diretamente mediante ``feature.FIELD`` ou feature.get("Field"). Por exemplo::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):
		layer = currentLayer()
		features = layer.features()
		for feature in features:
			print "ID: ", feature.ID, " NAME: ", feature.get("NAME")
			
A  saída por console será::

	ID:  1  NAME:  Feature1
	ID:  2  NAME:  Feature2
	ID:  3  NAME:  Feature2
	ID:  4  NAME:  Feature2
	ID:  5  NAME:  Feature3
	ID:  6  NAME:  Feature3
	ID:  7  NAME:  Feature3

	
Seleção
+++++++

Outro tipo de featureSet é :javadoc:`DefaultFeatureSelection <DefaultFeatureSelection>`. Faz referência aos objetos que temos selecionados na camada.

Um exemplo de seu uso, tendo 3 entidades selecionadas::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):
		layer = currentLayer()
		#features = layer.features()
		selection = layer.getSelection()

		for feature in selection:
			print "ID: ", feature.ID, " NAME: ", feature.get("NAME")

Dando como resultado::

	ID:  1  NAME:  Feature1
	ID:  2  NAME:  Feature2
	ID:  3  NAME:  Feature2

Temos dois métodos especiais para esta classe que são ``.selectAll()`` para selecionar todas as entidades da camada ou ``.deselectAll()`` para deselecionar todos.

Por exemplo, adicionaremos a seleção de certas entidades que cumpram um critério::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		layer = currentLayer()

		#Entidades da camada
		features = layer.features()

		#Seleccion de entidades 
		selection = layer.getSelection()
		selection.deselectAll()
		
		for feature in features:
			if feature.ID < 3:
				# Adicionamos as entidades na seleção
				selection.select(feature)

Se quisermos eliminar entidades da seleção, podemos usar o método ``.deselect(feature)``

Também podemos criar uma seleção ou várias::

	from gvsig import *

	def main(*args):

		# Cria a nova seleção
		layer = currentLayer()
		features = layer.features()
		newselection = layer.getDataStore().createSelection()
		
		for f in features:
			if f.ID!=10:
				newselection.select(f)

		layer.getDataStore().setSelection(newselection)
