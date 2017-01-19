Outras fontes de dados
======================

Neste capítulo está incluído diferentes funções interessantes para trabalhar com diferentes fontes de dados. Como caso general vamos introduzir a função ``openStore(servertype, parameters)`` para carregar fontes de dados a partir uma rota ou  link.

Se estes serviços já se encontram abertos em nossa Vista, podemos acessar no conteúdo de dados selecionando a camada e acessando-a, por exemplo, mediante um ``currentLayer()``.


PostgreSQL
----------

A partir do Scripting podemos abrir conexões para fontes de dados de tipo ``PostgreSQL`` por exemplo. A função para utilizar será ``openStore(servertype, parameters)``.


O exemplo que segue, é com uma base de dados PostGIS criada em local em uma máquina Ubuntu seguindo os seguintes passos (pode variar segundo versões ou distribuições)::

    sudo apt-get install postgresql postgresql-comtrib
    sudo apt-get install postgresql-9.5-postgis-2.2
    sudo apt-get install pgadmin3

    $ sudo su - postgres
    $ psql

    postgres=# select "version"();

    #Establecer comtraseña entrando en psql
    postgres=# \password

    user: postgres
    pass: postgres

    # Create new db and table
    $ dropdb -U postgres ej1
    $ createdb -U postgres ej1
    $ psql -U postgres ej1
    psql (9.5.4)
    Digite «help» para obtener ayuda.

    ej1=# create extension postgis;
    CREATE EXTENSION
    ej1=# create extension postgis_topology;
    CREATE EXTENSION
    ej1=# select postgis_full_version();
    ej1=# select srtext from spatial_ref_sys where srid=23030;
    ej1=# create table ciudades (gid serial PRIMARY KEY, nombre varchar, poblacion integer);
    CREATE TABLE
    ej1=# alter table ciudades add column geom geometry (PointZ, 23030);
    ALTER TABLE
    ej1=# \d ciudades
                                            Tabla «public.ciudades»
      Columna  |          Tipo          |                          Modificadores
    -----------+------------------------+------------------------------------------------------------------
     gid       | integer                | not null valor por omisión nextval('ciudades_gid_seq'::regclass)
     nombre    | character varying      |
     poblacion | integer                |
     geom      | geometry(PointZ,23030) |
    Índices:
        "ciudades_pkey" PRIMARY KEY, btree (gid)

    ej1=# insert into ciudades (nombre, poblacion, geom) values ('Ciudad A', 10000, st_geomfromtext('POINTZ(700000 45000000 10)', 23030));
    INSERT 0 1
    ej1=# insert into ciudades (nombre, poblacion, geom) values ('Ciudad B',50000, st_geomfromtext('POINT(720000 4470000 15)',23030));
    INSERT 0 1
    ej1=# select * from ciudadeS;
     gid |  nombre  | poblacion |                                geom
    -----+----------+-----------+--------------------------------------------------------------------
       1 | Ciudad A |     10000 | 01010000A0F659000000000000C05C2541000000002A7585410000000000002440
       2 | Ciudad B |     50000 | 01010000A0F65900000000000000F92541000000003C0D51410000000000002E40

No seguinte exemplo vemos como introduzir todos os parâmetros necessários para uma conexão PostgreSQL::

    from gvsig.utils import openStore

    def main(*args):
        os = openStore('PostgreSQL',port='5432',
                                    JDBCDriverClass='org.postgresql.Driver',
                                    UseSSL='false',
                                    Schema='public',
                                    Catalog='',
                                    URL='jdbc:postgresql://localhost/ej1',
                                    BaseOrder='',
                                    Workingarea=None,
                                    CRS='EPSG:23030',
                                    PKFields='gid',
                                    BaseFilter='',
                                    DefaultGeometryField='geom',
                                    Fields=None,
                                    Table='ciudades',
                                    SQL='',
                                    password='postgres',
                                    dbname='ej1',
                                    host='localhost',
                                    dbuser='postgres',
                                    ProviderName='PostgreSQL')

Uma vez feita a conexão, podemos comprovar que tipo de objeto que estamos tratando, se é um :javadoc:`DefaultFeatureStore <DefaultFeatureStore>`, e por tanto, podemos trabalhar da mesma forma que uma capa vectorial, já que o objeto é do mesmo tipo::

    print "** os: ", type(os)
    features = os.features()
    for f in features:
        print f.getValues()
                                
Produciendo por comsola una salida similar a::

    ** os:  <type 'org.gvsig.fmap.dal.feature.impl.DefaultFeatureStore'>
    {u'gid': 5, u'geom': POINT Z (720000.0 4600000.0 50.0), u'nombre': u'Ciudad A', u'poblacion': 30000}
    {u'gid': 6, u'geom': POINT Z (725000.0 4601000.0 10.0), u'nombre': u'Ciudad A', u'poblacion': 30000}
    {u'gid': 7, u'geom': POINT Z (725000.0 4651000.0 15.0), u'nombre': u'Ciudad A', u'poblacion': 30000}
    {u'gid': 8, u'geom': POINT Z (730000.0 4659000.0 20.0), u'nombre': u'Ciudad A', u'poblacion': 30000}
    {u'gid': 9, u'geom': POINT Z (722000.0 4620000.0 20.0), u'nombre': u'Ciudad A', u'poblacion': 30000}

Geojson
-------
Para carregar um arquivo Geojson fazendo uso de da extensão de GDAL::

    # encoding: utf-8

    import gvsig
    from gvsig.utils import openStore

    def main(*args):

        os = openStore('OGRDataStoreProvider',file="/home/osc/gvsig-devel/example-data/countries.geojson",
                                             CRS="EPSG:4326",
                                             comnectionString=None,
                                             layerName="OGRGeoJSON",
                                             defaultGeometryField=None,
                                             ignoreSpatialFilter=True,
                                             ProviderName="OGRDataStoreProvider"
                                             )
        print "geojson: ", type(os)
        features = os.features()
        for feature in features:
            print feature




Arquivo LIDAR: Las
------------------

Carregando arquivo LAS com o driver de Whitebox::

    # encoding: utf-8

    import gvsig
    from gvsig.utils import openStore

    def main(*args):

        os = openStore('WhiteboxLASDataStoreProvider', file="/home/osc/LiDAR_6429129715959308627.las",
                                                  thinningResolution=None,
                                                  CRS="EPSG:25830",
                                                  thinningDivisor=None,
                                                  ProviderName="WhiteboxLASDataStoreProvider")

        print os
        features = os.features()
        print "Las size: ", features.getSize()


Raster
------

Carregando arquivo tif::

    # encoding: utf-8

    import gvsig
    from gvsig.utils import openStore

    def main(*args):

        os = openStore('Gdal Store',alphaband0None=None,
                                    visible=None,
                                    crs="EPSG:4326",
                                    uri="file:/home/osc/Descargas/wc2.0_5m_srad/wc2.0_5m_srad_12.tif",
                                    rmf_folder="/home/osc/Descargas/wc2.0_5m_srad",
                                    frame=None,
                                    selected_option=0
                                    )

        print "raster type: ", type(os)

