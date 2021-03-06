Trabalhando com camadas: FLayer
================================

Trabalhando com camadas :javadoc:`FLayer <FLayer>`


.. py:function:: createShape(definition, [filename=None, geometryType=None, CRS=None, prefixname="tmpshp"])

   Creates a new shape layer. If geometryType is None, will take geometry type from the definition. If parameter geometryType and the geometry type inside the definition are different, raises an error.

    :param definition: Layer schema
	:type definition: DefaultFeatureType
    :param str filename: Full path
    :param geometryType: 
	:type geometryType: GeometryType
    :param str CRS: CRS Code
	:param str prefixname: first part of the temp name

A interface FLayer vem implementada em várias camadas de uso comum como são as camadas vetoriais ou raster.

Camadas vetoriais: FLyrVect
---------------------------

Uma das operações mais importante é o trabalho com camadas vetoriais :javadoc:`FLyrVect <FLyrVect>`. Procuramos simplificar o máximo possível o trabalho com elas, sobre todo para um novo desenvolvedor(programador).

Já vimos anteriormente a geração de esquemas que podem ser usados em camadas vetoriais. A única condição destes esquemas é que tenham um campo ``GEOMETRY`` com um tipo definido de geometria.

Criar camada vetorial
+++++++++++++++++++++++

Para a criação de camadas procuramos simplificar o máximo a função :py:func:``createShape(definition)``. O único parâmetro obrigatório será o esquema da camada com seu campo geometria.

Por exemplo, o mínimo necessário para criar uma camada é::


	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		schema = createFeatureType() # DefaultFeatureType

		schema.append("GEOMETRY", "GEOMETRY")
		schema.get("GEOMETRY").setGeometryType(POINT, D2)
		shape = createShape(schema)
                        
Desta forma, se a rota ou o nome do arquivo não é importante para nós, algo muito comum na hora de criar camadas para resultados de geoprocessos ou similares, a função se encarregará de atribuir uma rota em uma pasta temporária e um nome aleatório (baseado em códigos de tempo).

Outra forma seria usar diretamente a função `createLayer()`::

        layer = createLayer(schema=schema,
                            servertype="FilesystemExplorer",
                            layertype="Shape",
                            shpFile="/home/osc/temp/test1.shp",
                            CRS="EPSG:25830",
                            geometryType=geom.POINT
                            )

Assim no script anterior podemos adicionar as línhas::

    print "Nome: ", shape.getName()
    print "Rota: ", shape.getDataStore().getFullName()
	
Que dará como resultado o nome e rota da camada, similar a::

	Nome:  tmpshp-57afa1381035
	Rota:  C:\Users\Oscar\AppData\Local\Temp\tmp-andami\tmpshp-57afa1381035.shp
	
Se for necessário criar várias camadas e precisar diferenciá-las, podemos estabelecer o parâmetro ``prefixname`` o qual modificará a primeira parte do nome, mas seguirá criando todo o resto de rota temporária e assegurando-nos que será um nome único::


	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		schema = createFeatureType() # DefaultFeatureType

		schema.append("GEOMETRY", "GEOMETRY")
		schema.get("GEOMETRY").setGeometryType(POINT, D2)
		shape1 = createShape(schema, prefixname="resultado")
		shape2 = createShape(schema, prefixname="erros")
		shape3 = createShape(schema, prefixname="pontos")
		
		print "Nome: ", shape1.getName(), "\tRota: ", shape1.getDataStore().getFullName() 
		print "Nome: ", shape2.getName(), "\tRota: ", shape2.getDataStore().getFullName()
		print "Nome: ", shape3.getName(), "\tRota: ", shape3.getDataStore().getFullName()

No console nos mostra::

	Nome:  resultado-57afa4f612c9 	Rota:  C:\Users\Oscar\AppData\Local\Temp\tmp-andami\resultado-57afa4f612c9.shp
	Nome:  erros-57afa4f613a6 	Rota:  C:\Users\Oscar\AppData\Local\Temp\tmp-andami\erros-57afa4f613a6.shp
	Nome:  pontos-57afa4f61446 	Rota:  C:\Users\Oscar\AppData\Local\Temp\tmp-andami\pontos-57afa4f61446.shp
	
Modificar esquema de uma camada
+++++++++++++++++++++++++++++++

O seguinte script modificará o esquema de una camada. Para isto teremos que criar um novo esquema baseado na camada anterior, mediante ``createFeatureType(layer_schema)``, realizar as modificações e atualizar a camada::

	from gvsig import *
	from gvsig import geom

	def main(*args):
		"""Updating schema of existent layer"""
		
		layer = currentLayer()
		
		schema = layer.getSchema()
		newschema = createSchema(schema)
		newschema.append("ID2", "STRING")
		
		layer.edit()
		layer.update(newschema)
		layer.commit()

Operação com entidades
+++++++++++++++++++++++++

Una vez criada a nova camada ou acessada uma já existente com ``currentLayer()`` ou ``view.getLayer("Name")``, podemos acessar as suas entidades mediante o método ``.features()``, tal como explicamos no guia de Acesso a dados.

Em seguida o que faremos é adicionar dados a esta camada vetorial. Para isto colocamos a camada em modo de edição mediante ``layer.edit()`` e agregamos as entidades com ``layer.append(args)``::


	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		schema = createFeatureType() # DefaultFeatureType

		schema.append("ID", "INTEGER", 5)
		schema.append("NOME", "STRING", 10)
		schema.append("GEOMETRY", "GEOMETRY")
		schema.get("GEOMETRY").setGeometryType(POINT, D2)
		
		shape = createShape(schema, prefixname="resultado")

		
		print "Nome: ", shape.getName(), "\tRota: ", shape.getDataStore().getFullName()

		shape.edit()
		# Incluir dados na camada
		shape.append(ID=1, NOME="Valencia", GEOMETRY=createPoint2D(10, 10))
		# Dicionário
		shape.append({"ID": 2, "NOME": "Paris", "GEOMETRY":createPoint2D(15, 15)})
		shape.commit()

		currentView().addLayer(shape)


Outro exemplo adicionando entidades, usando a partir do Java::

	import gvsig
	reload(gvsig)
	from gvsig import *
	from gvsig import geom

	from org.gvsig.fmap.dal.feature import FeatureStore
	def main(*args):

		# Criar nova camada
		schema = createSchema()
		schema.append("ID", "INTEGER")
		schema.append("NOME", "STRING", 10)
		schema.append("GEOMETRY", "GEOMETRY")
		schema.get('GEOMETRY').setGeometryType(geom.POINT,geom.D2)

		camada = createShape(schema, prefixname="Camada_ponto")

		# Inserir com newfeature
		store = camada.getFeatureStore()

		newfeature = store.createNewFeature()
		newfeature.set("ID",1)
		newfeature.set("NOME","Feature1")
		newfeature.set("GEOMETRY", geom.createPoint(geom.D2, 1,2))

		camada.edit(FeatureStore.MODE_APPEND) #somente para camadas recém criadas
		store.insert(newfeature)
		camada.commit()

		# Inserir com append
		camada.edit()
		camada.append(ID=2,NOME='Feature2',GEOMETRY=geom.createPoint(geom.D2, 5, 3))

		camada.append({'ID':3,'NOME':'Feature2','GEOMETRY':geom.createPoint(geom.D2, 3, 3)})
		camada.append({'ID':4,'NOME':'Feature2','GEOMETRY':geom.createPoint(geom.D2, 2, 1)})
		camada.append({'ID':5,'NOME':'Feature3','GEOMETRY':geom.createPoint(geom.D2, 2, 6)})
		camada.append({'ID':6,'NOME':'Feature3','GEOMETRY':geom.createPoint(geom.D2, 6, 2)})
		camada.append({'ID':7,'NOME':'Feature3','GEOMETRY':geom.createPoint(geom.D2, 2, 7)})
		camada.commit()

		# Adicionar camada na vista
		currentView().addLayer(camada)

		print u"Informações das entidades"
		for l in camada.features():
		        print l 
		
Se ao final do script anterior adicionamos as seguintes linhas, veremos um exemplo para eliminar entidades::


		features = layer.features("ID < 6") #DefaultFeatureSet
		
		layer.edit()
		print type(layer)
		print features, type(features)
		for i in features:
			features.delete(i)

		layer.commit()

	
Para modificar os valores das entidades da nossa camada::

		layer.edit()

		for i in features:
			print i
			c = i.getEditable()
			c.set("NOME", "Modified_4")
			features.update(c)
			
		layer.commit()
		

É possível realizar copias de entidades (features) e depois modifica-las na sua camada original.

Exemplo: Extraímos certas entidades de una camada que tenha um Campo1 do tipo Long. Copiamos estas entidades para uma lista. Depois, modificamos estas entidades e voltemos a modificar sobre a camada inicial::

    from gvsig import *

    def main(*args):
        layer = currentLayer()
        features = layer.features('Campo1>2',sortBy="Campo1",asc=True)
        lista = []
        for f in features:
            print f
            copia = f.getCopy()
            print type(copia)
            lista.append(copia)

        print len(lista)
        layer.edit()
        for i in lista:
            value = i.get('Campo1')+0.01
            i = i.getEditable()
            i.set('Campo1', value)
            print "new value", i.get('Campo1'), type(i)
            featureSet = layer.features()
            layer.features().update(i)
        layer.commit()
