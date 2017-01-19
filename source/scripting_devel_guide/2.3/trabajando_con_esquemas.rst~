Trabalhando com esquemas: FeatureType
====================================

Os descritores contêm a informação sobre o nome, tipo, tamanho, precisão, etc. dos campos dos nossos dados.

Segundo seja o seu uso terá que cumprir certos requisitos. Por exemplo, se criamos uma capa vetorial, seu esquema tem que ter obrigatoriamente um campo **GEOMETRY** estabelecido.

Criando esquemas
----------------
.. note:: Esta é uma nota de advertência

    O nome de um campo em um documento shape não pode ter mais que 10 caracteres e também não é permitido utilizar caracteres especiais, espaços e acentuação. Underline, hifens, letras e números são permitidos.

.. py:function:: createFeatureType([schema=None]):
    
    Create an empty schema. If parameter schema is a :javadoc:`DefaultEditableFeatureType <DefaultEditableFeatureType>`, creates a copy of this schema :javadoc:`DefaultFeatureType <DefaultFeatureType>`.
	
    :param schema: Schema from other layer
    :type schema: :javadoc:`DefaultEditableFeatureType <DefaultEditableFeatureType>`
    :return: empty schema or a copy of a schema
    :type return: DefaultEditableFeatureType
	
A interface :javadoc:`DataTypes <DataTypes>` especifica um conjunto de constantes indicando os tipos de dados suportados por DAL.

Vamos a ver como criar um esquema com dois campos e atribuir este esquema em uma nova tabela::


	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		schema = createFeatureType() # DefaultFeatureType

		schema.append("ID", "INTEGER", 10)
		schema.append("NAME", "STRING", 20)

		dbf = createDBF(schema)
		
		d = loadDBF( dbf.getFullName()) # Carrega a tabela no  projeto
		
Também podemos criar um novo esquema baseado em outro já existente, muito útil para fazer copias de camadas. Para passar um esquema para a criação de camadas deve ser de tipo **DefaultFeatureType**, a função :py:func:``createFeatureType()`` converte o esquema de tipo :javadoc:`DefaultEditableFeatureType <DefaultEditableFeatureType>` para o que necessitamos::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		schema = createFeatureType() # DefaultFeatureType

		schema.append("ID", "INTEGER", 10)
		schema.append("NAME", "STRING", 20)

		dbf_1 = createDBF(schema)
		
		dbf_schema = dbf_1.getSchema() # DefaultEditableFeatureType

		# Novo esquema baseado em outra camada
		new_schema = createSchema(dbf_schema)
		new_schema.append("CODE", "STRING", 5)
		
		dbf_2 = createDBF(new_schema)


Una forma de acessar os descritores contidos no esquema é mediante ``schema.get("Campo")``. Também podemos iterar sobre eles. No seguinte exemplo, vemos a forma de adicionar diferentes campos de diferentes tipos em um novo esquema. Nós utilizamos uma função que itera sobre eles, para mostrar informações completas sobre o esquema que passamos como um parâmetro::

	from gvsig import *
	from gvsig.geom import *

	def showInfoFeatureType(schema):
		#Mostra o esquema
		attrSchema = schema.getAttrNames()
		print "Esquema attr: ", attrSchema
		#Primeiro campo do esquema
		field1 = schema.get(0)

		# Descrição dos campos
		print u"\nDescrição dos campos"
		for field in schema:
			print " Nome: ", field.getName(),
			print " \tDataTypeName: ", field.getDataTypeName(),
			print " \tType: ", field.getType(),
			print " \tSubType: ", field.getSubtype(),
			print u" \tPrecisão: ", field.getPrecision(),
			print " \tTamanho: ", field.getSize()
			
			if field.getDataTypeName() == 'Geometry':
			  geomType = field.getGeomType()
			  print " \tGeom Name: ", geomType.getName()
			  print " \tGeom FullName: ", geomType.getFullName()
			  print " \tType: ", geomType.getType()
			  print " \tSubType: ", geomType.getSubType()
			  print " \tGeometryClass: ", geomType.getGeometryClass()
			  print u" \tDimensão: ", geomType.getDimension()

	def main(*args):

		schema = createFeatureType()

		schema.append("ID", "INTEGER", 10)
		schema.append("NOME", "STRING", 20)
		schema.append("AREA", "DOUBLE", 20, 10)
		schema.append("DATA", "DATE", 20)
		schema.append("ATIVO", "BOOLEAN")
		schema.append("GEOMETRY", "GEOMETRY")
		schema.get('GEOMETRY').setGeometryType(POINT, D2)

		shape = createShape(schema, prefixname="date")
		currentView().addLayer(shape)
		showInfoFeatureType(schema)
		
No console veremos::

	Esquema attr:  [u'ID', u'NOME', u'AREA', u'DATA', u'ATIVO', u'GEOMETRY']

	Descrição dos campos
	 Nome:  ID  	DataTypeName:  Integer  	Type:  4  	SubType:  None  	Precisão:  0  	Tamanho:  10
	 Nome:  NOME  	DataTypeName:  String  	Type:  8  	SubType:  None  	Precisão:  0  	Tamanho:  20
	 Nome:  AREA  	DataTypeName:  Double  	Type:  7  	SubType:  None  	Precisão:  4  	Tamanho:  20
	 Nome:  DATA  	DataTypeName:  Date  	Type:  9  	SubType:  Date  	Precisão:  0  	Tamanho:  20
	 Nome:  ATIVO  	DataTypeName:  Boolean  	Type:  1  	SubType:  None  	Precisão:  0  	Tamanho:  0
	 Nome:  GEOMETRY  	DataTypeName:  Geometry  	Type:  66  	SubType:  Geometry  	Precisão:  0  	Tamanho:  0
		Geom Name:  Point2D
		Geom FullName:  Point:2D
		Type:  1
		SubType:  0
		GeometryClass:  <type 'org.gvsig.fmap.geom.jts.primitive.point.Point2D'>
		Dimensão:  2

Modificando esquemas
--------------------

No seguinte exemplo, vamos modificar um esquema::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		schema = createFeatureType() # DefaultFeatureType

		schema.append("ID", "INTEGER", 10)
		schema.append("NOME", "STRING", 20)
		schema.append("CODE", "STRING", 2)
		
		
		# Por índice
		schema.remove(0)
		print "Remove descritor ID: ", schema.getAttrNames()

		# Por descritor
		rm = schema.getEditableAttributeDescriptor("CODE")
		schema.remove(rm)
		print "Remove descritor CODE: ", schema.getAttrNames()

		# Adiciona o campo geometria
		schema.append("GEOMETRY", "GEOMETRY")
		schema.get("GEOMETRY").setGeometryType(POINT, D2)
		print " Adiciona o campo geometria: ", schema.getAttrNames()
		
Mostra no console o seguinte::

	Remove descritor ID:  [u'NOME', u'CODE']
	Remove descritor CODE:  [u'NOME']
	Adiciona o campo geometria:  [u'NOME', u'GEOMETRY']
		
		
Esquema para camadas vetoriais
------------------------------

Para a criação de uma camada vetorial, será similar ao exemplo, que deverá ter somente um campo ``GEOMETRY`` do tipo ``GEOMETRY``. Depois de criar este campo, teremos que definir qual o tipo de geometria. 


No exemplo mostramos o caso típico de criação de uma nova camada vetorial::

	from gvsig import *
	from gvsig.geom import *

	def main(*args):

		schema = createFeatureType() # DefaultFeatureType

		schema.append("ID", "INTEGER", 10)
		schema.append("NOME", "STRING", 20)
		schema.append("GEOMETRY", "GEOMETRY")
		schema.get("GEOMETRY").setGeometryType(POINT, D2)

		shape = createShape(schema)
		currentView().addLayer(shape)
		

