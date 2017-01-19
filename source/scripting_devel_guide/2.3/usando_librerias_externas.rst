Usando bibliotecas externas
===========================

GDAL
----
Foi incluída uma das aplicações mais comuns de uso com GDAL que se denomina ogr2ogr. Para usá-la a partir de u, script podemos usar as seguintes linhas, modificando os caminhos pelos de nossos dados. O resultado é uma transformação de um arquivo geojson em um shape::

	from gvsig import *

	from gvsig import uselib
	uselib.use_plugin("org.gvsig.gdal.app.mainplugin")
	from org.gvsig.gdal.app.mainplugin.common import ogr2ogr


	def gdalapp(argsAsString):
		import shlex
		x = shlex.split(argsAsString)
		print "Function: ", x.pop(0)
		print "Args: ", x
		ogr2ogr.main(x)

	def main(*args):

		"""ogr2ogr from gdal"""
		"""
		ogr2ogr.main(["-t_srs", "CRS:84", 
					  "-f", 
					  "ESRI Shapefile", 
					  "C:/temp/j1", 
					  "C:/temp/countries.geojson",
					  "-overwrite",
					  "-skipfailures"
					  ])
		"""

		
		gdalapp("ogr2ogr -t_srs 'EPSG:4326' -f 'ESRI Shapefile' C://temp//j1.shp D://gvdata//countries023.geojson -overwrite -skipfailures")
		loadShapeFile("C:/temp/j1.shp")

jfreechart
----------
Uma biblioteca que vem com o gvSIG é a jfreechart e podemos usá-la para gerar gráficos.

Gerar gráfico e salvá-lo em formato ``.jpeg`` no disco.

.. figure::  images/pieChart.jpeg
   :align:   center
   
Código::

	from java.io import File
	from org.jfree.chart import ChartUtilities
	from org.jfree.chart import ChartFactory
	from org.jfree.chart import JFreeChart
	from org.jfree.data.general import DefaultPieDataset

	#Save chart into jpeg file

	def main():
		  dataset = DefaultPieDataset( )
		  dataset.setValue("IPhone 5s", float( 20 ) )
		  dataset.setValue("SamSung Grand", float( 20 ) )
		  dataset.setValue("MotoG", float( 40 ) )
		  dataset.setValue("Nokia Lumia", float( 10 ) )

		  chart = ChartFactory.createPieChart(
			 "Mobile Sales", # chart title
			 dataset, # data
			 True, # include legend
			 True,
			 False)
			 
		  width = 640
		  height = 480
		  pieChart = File( "D:/pieChart.jpeg" )
		  ChartUtilities.saveChartAsJPEG( pieChart , chart , width , height )
	   

Outro exemplo que gera uma tela com o resultado.

.. figure::  images/simple_pie_chart.png
   :align:   center
   
Código::

	from org.jfree.chart import ChartFactory 
	from org.jfree.chart import ChartFrame 
	from org.jfree.chart import JFreeChart
	from org.jfree.data.general import DefaultPieDataset
	from org.jfree.ui import RefineryUtilities
	"""
	* A simple introduction to using JFreeChart. This demo is described in the
	* JFreeChart Developer Guide.
	* Translated from Java to Jython by Alfonso Reyes
	"""
	class First:
		"""
		* The starting point for the demo.
		*
		* @param args ignored.
		"""
		# create a dataset...
		data = DefaultPieDataset()
		data.setValue("Category 1", 43.2)
		data.setValue("Category 2", 27.9)
		data.setValue("Category 3", 79.5)
		# create a chart...
		chart = ChartFactory.createPieChart(
			"Sample Pie Chart",
			data,
			True, # legend?
			True, #tooltips?
			False # URLs?
		)
		# create and display a frame...
		
		frame = ChartFrame("First", chart)
		#frame.setSize(100 , 100) #Position
		#RefineryUtilities.centerFrameOnScreen( frame )
		frame.pack()
		frame.setVisible(True)

	def main():
		app = First()
		
		
jOpenDocument
-------------

Editando ODT
++++++++++++

Abrir novo documento do LibreOffice

jOpenDocument - Insert Field - fieldName

Inserir uma imagem
Clique direito sobre a Imagem: Imagem - Opções - Nome: Imagem1


Editando ODS
++++++++++++

.. note::

    Después de modificarlo hay que abrirlo y presionas ``Control+Mayusculas+F9`` para recalcular las celdas

Exemplos
++++++++

Editar Planilha ODS::

    from gvsig import *

    from java.io import File
    from java.util import Date

    from org.jopendocument.model import OpenDocument
    from org.jopendocument.dom.spreadsheet import SpreadSheet
    from org.jopendocument.dom import OOUtils

    def main(*args):
        #Exemplo de Edição de arquivos ODS con jOpenDocument
        
        #baseado en http://www.jopendocument.org/start_spreadsheet_2.html
        #arquivos modelo http://www.jopendocument.org/downloads.html

        pathTemplate = r"C:/joo/invoice.ods"
        pathOutput = r"C:/joo/fillingTest1.ods"

        #Acesso a planilha e número de folha
        file = File(pathTemplate)
        sheet = SpreadSheet.createFromFile(file).getSheet(0)

        #Definir data atual à célula I10
        sheet.getCellAt("I10").setValue(Date())

        #Modificar o valor da célula 1,1. Seria B2
        sheet.setValueAt("Template - 1", 1, 1)

        #Várias modificações
        sheet.getCellAt("B27").setValue("On site support")
        sheet.getCellAt("F24").setValue(301)
        sheet.getCellAt("H27").setValue(350)
        sheet.getSpreadSheet().getTableModel("Products").setValueAt(10, 5, 4) #F27

        #Salvamos o arquivo
        outputFile = File(pathOutput)
        OOUtils.open(sheet.getSpreadSheet().saveAs(outputFile))


Editar Planilha ODT::

    from gvsig import *
    import sys
    from geom import *

    from java.io import File
    #from org.jdom import Namespace
    from java.util.Map import *
    import java.util.ArrayList as ArrayList

    from org.jopendocument.dom.template import JavaScriptFileTemplate
    from org.jopendocument.util.CollectionUtils import createMap

    def main(*args):
        #Exemplo de Edição de arquivos ODT com jOpenDocument
        
        #baseado en http://www.jopendocument.org/start_text_2.html
        #arquivos modelo http://www.jopendocument.org/downloads.html

        pathTemplate = r"C:\joo\test.odt"
        pathOutput = r"C:\joo\test4"
        
        #Criamos modelo
        templateFile = File(pathTemplate)
        outFile = File(pathOutput)
        template = JavaScriptFileTemplate(templateFile)

        # Principal: Determinar valores de um campo
        template.setField("toto", "value set using setField()")
        template.setField("a", "14")
        #Ao dar duplo clique sobre a linha verde aparece o nome 
        #ao qual temos que fazer referência
        template.hideParagraph("p1") #showParagraph
        template.showSection("section1") #hideSection


        #Valores da tabela
        months = ArrayList()
        months.add(createMap("name","January",   "min", "-12", "max", "3"))
        months.add(createMap("name", "February", "min", "-8",  "max", "5"))
        months.add(createMap("name", "March", "min", "-5", "max", "12"))
        months.add(createMap("name", "April", "min", "-1", "max", "15"))
        months.add(createMap("name", "May", "min", "3", "max", "21"))
        template.setField("months", months)
        
        ddoc = template.createDocument()
        ddoc.saveAs(outFile)


Substituir imagem::

    pathImg = r"C:/joo/img02.png"
    ddoc = template.createDocument()
    #ddoc.getDescendantByName("draw:frame","Imagen1").setAttribute("href", "file:///" + pathImg,Namespace.getNamespace("xlink", "http://www.w3.org/1999/xlink"))  
    ddoc.saveAs(outFile)
	
	
