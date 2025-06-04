# maps


import pandas as pd
import geopandas as gp
import matplotlib as mpl
import matplotlib.pyplot as plt


# Obtencion de mapas
mundo = gp.read_file(gp.datasets.get_path('naturalearth_lowres'))
mundo.dtypes
mundo.head


ciudades = gp.read_file(gp.datasets.get_path('naturalearth_cities'))
ciudades.dtypes
ciudades.head


ny = gp.read_file(gp.datasets.get_path('nybb'))
ny.dtypes
ny.head


pais = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_0.shp")
pais


provincias = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_1.shp")
provincias



cantones = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_2.shp")
cantones


limon = cantones.loc[cantones.NAME_1 == "Limón", :]
limon



#graficar mapa 

mundo = gp.read_file(gp.datasets.get_path('naturalearth_lowres'))
fig, ax = plt.subplots()
mundo.plot(ax = ax)
plt.show()

pais = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_0.shp")

fig, ax = plt.subplots()
no_plot = ax.set_title("Mapa de Costa Rica")
pais.plot(ax = ax)
plt.show()


provincias = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_1.shp")

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Mapa de las provincias de Costa Rica")
provincias.plot(ax = ax, color = 'white', edgecolor = 'black')
plt.show()


cantones = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_2.shp")

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Mapa de los cantones de Costa Rica")
no_plot = cantones.plot(ax = ax, column = "NAME_1", legend = True, cmap = 'Dark2')

leg = ax.get_legend()
leg.set_title("Provincias")
leg._set_loc((1.05, 0.5))

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()


limon = cantones.loc[cantones.NAME_1 == "Limón", :]

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-84, -82.5)
no_plot = ax.set_ylim(9, 11)
no_plot = ax.set_title("Mapa de los cantones de Limón")
no_plot = limon.plot(ax = ax, column = "NAME_2", legend = True, cmap = 'Dark2')

leg = ax.get_legend()
leg.set_title("Cantones")
leg._set_loc((0.9, 0.5))

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()


limon = cantones.loc[cantones.NAME_1 == "Limón", :]

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-84, -82.5)
no_plot = ax.set_ylim(9, 11)
no_plot = ax.set_title("Mapa de los cantones de Limón")
no_plot = limon.plot(ax = ax, column = "NAME_2", legend = True, cmap = 'Dark2')

leg = ax.get_legend()
leg.set_title("Cantones")
leg._set_loc((0.9, 0.5))

no_plot = fig.text(0.37, 0.91, "Datos actualizados al 16 de julio, 2022", fontdict = {"fontsize":7})
no_plot = fig.text(0.6, 0.02, "Fuente: GADM", fontdict = {"fontsize":6})

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()


#Los datos de los mapas que obtuvimos anteriormente corresponden únicamente a información geográfica y, aunque, es información muy interesante en ocasiones vamos a querer generar un gráfico de mapa con base a un #valor de una columna de nuestra organización o de nuestro interés. Para ello, debemos conocer tecnicas de agrupación y combinación como se muestra en el siguiente ejemplo:


#Se utilizará la tabla de delitos la cual contiene datos del OIJ sobre reportes de homicidios en Costa Rica durante el año 2024.

cantones = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_2.shp")

delitos = pd.read_csv("../../../datos/delitos_cr_v1.csv")
delitos.Fecha = pd.to_datetime(delitos.Fecha)
delitos


#Queremos graficar en un mapa la cantidad de homicidios por cantón para esto primero vamos a agrupar los datos por cantón y luego contar cuantos reportes hay en cada uno, para esto usaremos la función groupby #del objeto pandas.DataFrame
delitos_canton = delitos.groupby(["Canton"]).count()["Delito"]
delitos_canton.head()

#Al ser dos tablas de diferente procedencia, puede haber muchos nombres de cantones que no coincidan debido a que utilicen un formato diferente. Por tanto, es importante limpiar el texto y estandarizarlo.

import re
def limpiar_texto(texto):
  texto = texto.upper()
  texto = re.sub('Á','A', texto)
  texto = re.sub('É','E', texto)
  texto = re.sub('Í','I', texto)
  texto = re.sub('Ó','O', texto)
  texto = re.sub('Ú','U', texto)
  texto = re.sub('Ñ','N', texto)
  return texto

cantones.NAME_2 = [limpiar_texto(x) for x in cantones.NAME_2]
delitos_canton.index = [limpiar_texto(x) for x in delitos_canton.index]

set(delitos_canton.index) - set(cantones.NAME_2)


#{'PUERTO JIMENEZ', 'VASQUEZ DE CORONADO'}
#Puerto Jimenez es un cantón nuevo, antes formaba parte de golfito. Por tanto, se renombra PUERTO JIMENEZ por GOLFITO.

#El cantón VASQUEZ DE CORONADO esta mal escrito, debe ser: VAZQUEZ DE CORONADO.


delitos_canton.index = [re.sub('VASQUEZ DE CORONADO','VAZQUEZ DE CORONADO', x) for x in delitos_canton.index]

delitos_canton[delitos_canton.index == "GOLFITO"] = delitos_canton[delitos_canton.index == "GOLFITO"][0] + delitos_canton[delitos_canton.index == "PUERTO JIMENEZ"][0]
no_print = delitos_canton.pop("PUERTO JIMENEZ")

set(delitos_canton.index) - set(cantones.NAME_2)



#Tenemos 2 tablas y necesitamos unirlas para poder realizar el gráfico, para esto utilizaremos la función merge del objeto pandas.DataFrame. Los parámetros que se pueden utilizar son los siguientes:

#right: tabla con la que se va a unir.
#left_on: variable de la tabla izquierda que se usará para la unión.
#right_on: variable de la tabla derecha que se usará para la unión.
#right_index: Si se debe usar el indice de la tabla como variable para la unión.
#left_index: Si se debe usar el indice de la tabla como variable para la unión.


delito_cantones = cantones.merge(delitos_canton, left_on = "NAME_2", right_index = True, how = 'left')
delito_cantones = delito_cantones.fillna(0)
delito_cantones



fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Cantidad de asaltos por cantón en Costa Rica (2017)")
mapa = delito_cantones.plot(ax = ax, column = "Delito", legend = True, cmap = 'OrRd')

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()



#A lo largo del año 2020 nos vimos inundados de gráficos y mapas relacionados con el covid-19, hoy vamos a utilizar el siguiente gráfico como ejemplo para ilustrar algunos de los problemas que podemos encontrar #al crear este tipo de mapas.


#Carga de datos
cantones = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_2.shp")

covid = pd.read_csv("../../../datos/covid_2021_v1.csv", sep = ";")
covid = covid.loc[covid.fecha == "2020-09-11", :] # Se filtran para la fecha correspondiente.
covid

#Limpieza de datos
cantones.NAME_2 = [limpiar_texto(x) for x in cantones.NAME_2]
covid.canton = [limpiar_texto(x) for x in covid.canton]

set(covid.canton) - set(cantones.NAME_2)

#{'OTROS'}
#Combinar
covid_cantones = cantones.merge(covid, left_on = "NAME_2", right_on = "canton", how = 'left')
covid_cantones = covid_cantones.fillna(0)
covid_cantones


#Como vemos en el gráfico original, los cantones están agrupados en una serie de intervalos, utilizaremos un ciclo for para replicar estos grupos.

grupos = []

for i in covid_cantones.casos:
    if i == 0 :
      grupos.append("0")
    elif i > 1 and i <= 4:
      grupos.append("1-4")
    elif i > 4 and i <= 9:
      grupos.append("5-9")
    elif i > 9 and i <= 14:
      grupos.append("10-14")
    elif i > 14 and i <= 20:
      grupos.append("15-20")
    elif i > 20:
      grupos.append("20 o más")
    else:
      grupos.append("Sin Datos")
      
print(grupos)

covid_cantones["grupos"] = pd.Categorical(
  values = grupos, 
  categories = ["0", "1-4", "5-9","10-14","15-20","20 o más"],
  ordered = True)

#Graficar
fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Casos activos de Covid-19, Costa Rica setiembre 2021")

covid_cantones.plot(ax = ax, column = "grupos", legend = True, edgecolor = 'black', cmap = 'OrRd')

leg = ax.get_legend()
leg.set_title("Casos")
leg._set_loc((1, 0.5))

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()

#Volvamos a hacer estos grupos usando grupos igualmente espaciados pero con intervalos más realistas, para esto utilizaremos la función cut de la biblioteca pandas.

covid_cantones.grupos = pd.cut(x = covid_cantones.casos, bins = 6)

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Casos activos de Covid-19, Costa Rica setiembre 2021")

covid_cantones.plot(ax = ax, column = "grupos", legend = True, edgecolor = 'black', cmap = 'OrRd')

leg = ax.get_legend()
leg.set_title("Casos")
leg._set_loc((0.90, 0.55))

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()

#Modifiquemos las etiquetas para hacer el gráfico más sencillo de leer.

covid_cantones.grupos = pd.cut(
  x = covid_cantones.casos, bins = 6,
  labels = ["menos de 946", "946 - 1890", "1890 - 2834", "2834 - 3778", "3778 - 4722", "más de 4722"])

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Casos activos de Covid-19, Costa Rica setiembre 2021")

covid_cantones.plot(ax = ax, column = "grupos", legend = True, edgecolor = 'black', cmap = 'OrRd')

leg = ax.get_legend()
leg.set_title("Casos")
leg._set_loc((0.90, 0.55))

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()

#En esta estrategia se buscan grupos igualmente poblados es decir que cada grupo tenga la misma cantidad de regiones, para esto utilizaremos la función qcut de la biblioteca pandas.

covid_cantones.grupos = pd.qcut(x = covid_cantones.casos, q = 6)

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Casos activos de Covid-19, Costa Rica setiembre 2021")

covid_cantones.plot(ax = ax, column = "grupos", legend = True, edgecolor = 'black', cmap = 'OrRd')

leg = ax.get_legend()
leg.set_title("Casos")
leg._set_loc((0.90, 0.55))

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()

#Modifiquemos las etiquetas para hacer el gráfico más sencillo de leer.

covid_cantones.grupos = pd.qcut(
  x = covid_cantones.casos, q = 6, 
  labels = ["menos de 28","28 - 79","79 - 156", "156 - 217", "217 - 468", "más de 468"])

fig, ax = plt.subplots()
no_plot = ax.set_xlim(-86, -82.5)
no_plot = ax.set_ylim(8, 11.5)
no_plot = ax.set_title("Casos activos de Covid-19, Costa Rica setiembre 2021")

covid_cantones.plot(ax = ax, column = "grupos", legend = True, edgecolor = 'black', cmap = 'OrRd')

leg = ax.get_legend()
leg.set_title("Casos")
leg._set_loc((0.90, 0.55))

no_plot = plt.axis('off') # Eliminar ejes
plt.tight_layout()
plt.show()

#Al trabajar con mapas estáticos tenemos una gran limitante a la hora de colocar etiquetas o presentar mucha información por suerte la mayoría de los usuarios hoy en dia interactuan con los gráficos a través de #dispositivos que soportan gráficos interactivos.

#Para trabajar con gráficos interactivos vamos a utilizar adicionalmente el paquete folium:

#pip install folium
#Luego podemos cargar el paquete de la siguiente forma:

#Cuando cargamos un mapa con el paquete GeoPandas tenemos acceso también a gráficos interactivos que en el fondo los genera con folium. Para ello debemos seguir los siguientes pasos:

#Obtener el centro de nuestro mapa.
cantones = gp.read_file("../../../datos/cr_map_v1/gadm41_CRI_2.shp")

x = cantones.centroid.x.mean()
y = cantones.centroid.y.mean()

#Generar el mapa.
m = folium.Map(
  location = [y, x],         # Punto inicial
  zoom_start = 9,            # Zoom inicial
  tiles = "cartodbpositron" # Diseño del mapa.
  # otros diseños son: openstreetmap, mapquestopen, MapQuest Open Aerial, 
  # Mapbox Bright, Mapbox Control Room, stamenterrain, stamentoner, 
  # stamenwatercolor, cartodbpositron, cartodbdark_matter
)
m
#Dibujar los datos de nuestro mapa.
mapa = cantones.explore(m = m)
mapa

#agregar color 

m = folium.Map(
  location = [y, x],
  zoom_start = 9,
  tiles = "cartodbpositron"
)

mapa = cantones.explore(
  m = m,
  column  = "NAME_1",                 # Realiza un mapa de color en base a la columna.
  tooltip = ["NAME_1", "NAME_2"],     # Columnas a mostrar en el tooltip.
  popup   = ["NAME_1", "NAME_2"],     # Columnas a mostrar en el popup.
  cmap    = "Set1",                   # Conjunto de colores a utilizar en el mapa de color.
  style_kwds = dict(color = "black")) # Color de la linea de división.

mapa


#agregar marcadores 

peligrosos = delito_cantones.sort_values(by = ['Delito'], ascending=False)
peligrosos = peligrosos.iloc[0:5, :]

peligrosos['coords'] = peligrosos['geometry'].apply(lambda x: x.representative_point().coords[:])
peligrosos['coords'] = [coords[0] for coords in peligrosos['coords']]

m = folium.Map(
  location = [y, x],
  zoom_start = 9,
  tiles = "cartodbpositron"
)

mapa = delito_cantones.explore(
  m = m,
  column  = "Delito",                       # Realiza un mapa de color en base a la columna.
  tooltip = ["NAME_1", "NAME_2", "Delito"], # Columnas a mostrar en el tooltip.
  popup   = ["NAME_1", "NAME_2", "Delito"], # Columnas a mostrar en el popup.
  cmap    = "Reds",                         # Conjunto de colores a utilizar en el mapa de color.
  style_kwds = dict(color = "black"))       # Color de la linea de división.

for idx, row in peligrosos.iterrows():
  no_plot = mapa.add_child(
    folium.Marker(
      location = row['coords'][::-1],
      popup = "Provincia: " + row['NAME_1'] + '<br>' +
              "Cantón: " + row['NAME_2'] + '<br>' +
              "Homicidios: " + str(row['Delito']),
      icon = folium.Icon(color = "red")
    )
  )

mapa

