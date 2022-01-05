OBJETIVO:
> Extraer el precio,descripción y ubicación geográfica de bienes inmuebles de `www.fincaraiz.com.co` 

CREADO POR: SEAN DAZA
`https://github.com/seandaza` 

> Ultima Actualización 13 de Noviembre de 2021







```python
import pandas as pd
import re
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.chrome.options import Options

opts = Options()
opts.add_argument("user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/71.0.3578.80 Chrome/95.0.4638.54 Safari/537.36")
driver = webdriver.Chrome('./chromedriver.exe', chrome_options=opts) 


#Numero de paginas a scrapear y url a escrapear:
number = 1
url_feed = 'https://www.fincaraiz.com.co/finca-raiz/venta/bogota?tipo=apartamentos-apartaestudios-casas-casascampestres-fincas-lotes-edificios'


#Acumulador de datos de interes por inmueble
total_cards = []


while number > 0:
    
    #Concatenador de paginas para recorrido Horizontal del Scrapping
    url = url_feed+str('?pagina=')+str(number)
    driver.get(url)
    inmuebles = driver.find_elements_by_xpath('//*[@id="listingContainer"]')
    inmueble = inmuebles[0]

    
    def cards(m,n):
        '''
        Extrae los datos de cada inmueble plasmados
        en cada tarjeta por recorrido horizontal.
        recibe como parametro a m y n que corresponden
        a los recorridos que hara el bucle,para omiir
        las publicidades.
        '''
        for i in range(m,n):
            i = str(i)

            precio = inmueble.find_element_by_xpath('.//div[1+"'+i+'"]/article/a/div[2]/div[1]/span[1]').text
            print('precio: ',precio)
            descripcion = inmueble.find_element_by_xpath('.//div[1+"'+i+'"]/article/a/div[2]/div[2]/span').text
            print('descripcion: ',descripcion)
            area = str(re.findall(r'\S{1,10}m²',descripcion, flags=re.M)).replace('[\'','').replace('\']','').replace('m²','').replace('[','').replace(']','')
            print('area: ',area)
            habitaciones = str(re.findall(r'•.+ha',descripcion, flags=re.M)).replace('[\'','').replace('\']','').replace('•','').replace('ha','').replace('[','').replace(']','')
            print('habitaciones: ',habitaciones)
            banos = str(re.findall(r'.\S+ba',descripcion, flags=re.M)).replace('[\'','').replace('\']','').replace('•','').replace('ba','').replace('[','').replace(']','')
            print('banos: ',banos)
            parqueaderos = str(re.findall(r'.\S+pa',descripcion, flags=re.M)).replace('[\'','').replace('\']','').replace('•','').replace('pa','').replace('[','').replace(']','')
            print('parqueaderos: ',parqueaderos)
            ubicacion = inmueble.find_element_by_xpath('.//div[1+"'+i+'"]/article/a/div[2]/div[3]').text
            print('ubicacion: ',ubicacion)
            link = inmueble.find_element_by_xpath('.//div[1+"'+i+'"]/article/a').get_attribute('href')
            print('link:',link)

            #internal route:
            
            opts = Options()
            opts.add_argument("user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/71.0.3578.80 Chrome/95.0.4638.54 Safari/537.36")
            driver2 = webdriver.Chrome('./chromedriver.exe', chrome_options=opts) 

            #Extraccion a profundidad por inmueble (Recorrido vertical de Scrapping)
            driver2.get(link)
            #delay = 10
        

            soup = driver2.find_element_by_xpath('//*[@id="general"]').text
            print('soup',soup)      
            descripcion = str(re.findall(r'Descripción general+\n+.+',soup, flags=re.M)).replace('[\'','').replace('\']','').replace('Descripción general','').replace('\\n','')
            print('descripcion: ',descripcion)
            areaConstruida = str(re.findall(r'Área construída+\n+.+',soup, flags=re.M)).replace('[\'','').replace('\']','').replace('Área construída','').replace('\\n','').replace('[','').replace(']','').replace('m²','')
            print('areaConstruida: ',areaConstruida)
            estrato = str(re.findall(r'Estrato+\n+.+',soup, flags=re.M)).replace('[\'','').replace('\']','').replace('Estrato','').replace('\\n','').replace('[','').replace(']','')
            print('estrato: ',estrato)
            administracion = str(re.findall(r'Administración+\n+.+',soup, flags=re.M)).replace('[\'','').replace('\']','').replace('Administración','').replace('\\n','').replace('[','').replace(']','').replace("$",'').replace('COP','')
            print('administracion: ',administracion)
            estado = str(re.findall(r'Estado+\n+.+',soup, flags=re.M)).replace('[\'','').replace('\']','').replace('Estado','').replace('\\n','').replace('[','').replace(']','')
            print('estado: ',estado)
            areaPrivada = str(re.findall(r'Área privada+\n+.+',soup, flags=re.M)).replace('[\'','').replace('\']','').replace('Área privada','').replace('\\n','').replace('[','').replace(']','').replace('m²','')
            print('areaPrivada: ',areaPrivada)
            antiguedad = str(re.findall(r'Antigüedad+\n+.+',soup, flags=re.M)).replace('[\'','').replace('\']','').replace('Antigüedad','').replace('\\n','').replace('[','').replace(']','')
            print('antiguedad: ',antiguedad)
            total_cards.append({
                "precio": precio,
                "area(m2)": area,
                "habitaciones": habitaciones,
                "banos": banos,
                "parqueaderos": parqueaderos,
                "ubicacion": ubicacion,
                "url": link,
                "descripcion": descripcion,
                "areaConstruida": areaConstruida,
                "estrato": estrato,
                "administracion": administracion,
                "estado": estado,
                "areaPrivada": areaPrivada,
                "antiguedad": antiguedad
            })
            
            #delay = 10
            driver2.close()
            
    # Filtrando las publicidades y los inmuebles invalidos
    cards(0,4)
    cards(7,13)
    cards(15,20)
    cards(23,46)
    number -= 1

driver.close()

#Construyendo el Dataframe
df = pd.DataFrame(total_cards)
print(df)
df.to_csv("finca_raiz.csv")

```


