Source code: https://colab.research.google.com/drive/1NMgoSuYol0JlHi4kFbxt_o98FMCml7Y8#scrollTo=S0ptTMNZCaBZ

## Introducción
Spotter AI es una startup centrada en la seguridad y la eficiencia. Aprovechamos la tecnología de código QR más avanzada para realizar validaciones de usuario sin problemas, garantizando un proceso de entrada rápido y seguro.

## Objetivo y contexto
Para este análisis se trabajará con un dataset del estacionamiento de la Universidad del Norte Santo Tomás de Aquino (UNSTA), en la sede campus. Se utilizarán modelos de aprendizaje automático de regresión y el método de Montecarlo con el fin de determinar la cantidad de autos que habrá en el estacionamiento dados ciertos parámetros.
Por motivos obvios de seguridad, no contamos con el dataset original del estacionamiento de la universidad. Es por esto que para el análisis primero tenemos que generar el mismo, con sus restricciones correspondientes.


# Dataset
Creamos el dataset principal con aquellos atributos que vayan a ser de relevancia para el modelo. Estos atributos son:
* Patente del vehículo
* Fecha y hora
* Día de la semana
* Temperatura
* Acción del vehículo (entrada o salida)

## Temperatura

Para este DataFrame se utilizó como base el dataset generado por el proyecto UNSTA Weather, el cual recopiló diferentes factores meteorológicos, una vez por hora, entre el 26/03/2019 y el 31/05/2020. En este caso, solo es pertinente conocer el valor de la temperatura.
Utilizando los valores del dataset como referencia, se expandió el mismo hasta el 30/11/2023, replicando los valores de temperatura cuando los valores de hora, día y mes son iguales, agregando un valor de ruido de 5 grados a los mismos para crear una leve variación con respecto a los datos del dataset original.

Se iteran todas las horas del rango de fechas que se desea anexar al DataFrame original y, utilizando la función get_temperature(date), se asigna un valor de temperatura a dicha hora.

```python
df = dataframe[['Datetime','Temp_Out']].copy()
df['Datetime'] = pd.to_datetime(df['Datetime'])
df.set_index('Datetime', inplace=True)
start_date = pd.to_datetime('2020-06-01 00:00:00')
end_date = pd.to_datetime('2023-11-30 23:00:00')
new_dates = pd.date_range(start_date, end_date, freq='H')
new_df = pd.DataFrame(index=new_dates)
new_df['Temp_Out'] = new_df.index.map(get_temperature)
extended_df = pd.concat([df, new_df])
extended_df.reset_index(inplace=True)
extended_df.rename(columns={"index": "Datetime", "Temp_Out": "Temperature"}, inplace=True)
df_temp = extended_df
```

![[Pasted image 20231211211040.png | 300]]

#### Generación dataset del estacionamiento

Para este DataFrame se utilizaron una mezcla de datos generados aleatoriamente y el DataFrame de temperatura generado anteriormente. Todos los vehículos tendrán una cantidad par de registros, puesto que necesitan al menos uno de entrada y uno de salida.

La frecuencia de entrada de los vehículos no es aleatoria, sino que está determinada por 4 factores:
* Hora
* Día de la semana
* Mes
* Temperatura al momento de ingresar

## Patentes

Para generar las patentes se tuvieron en cuenta los dos formatos de pantente actualente vigentes en Argentina (AAA000 y AA000AA). La posibilidad de que la función devuelva uno u otro formato es indistinta para ambos casos, y no se tuvo en cuenta el hecho de que también se pueden generar patentes aún inexistentes, puesto que esto no presenta ningún efecto en el análisis posterior.
```python
def generate_license_plate():
    format = random.choice([1, 2])
    if format == 1:
        # Devuelve una patente con formato AA000AA
        return ''.join(random.choices(string.ascii_uppercase, k=2)) + ''.join(random.choices(string.digits, k=3)) + ''.join(random.choices(string.ascii_uppercase, k=2))
    else:
        # Devuelve una patente con formato AAA000
        return ''.join(random.choices(string.ascii_uppercase, k=3)) + ''.join(random.choices(string.digits, k=3))
```
La función generate_datetime() srirve para generar una fecha aleatoria con ciertas restricciones definidas, y es la función más importante, puesto que esta es la que determina la frecuencia de entrada de los vehículos. Analicemos cada uno de los factores que influyen en la dicha frecuencia.
*   Mes: Cada mes tiene un valor específico de probabilidad de aparecer en la fecha generada. El mes con la mayor probabilidad de aparecer es abril, mientras que el que menos tiene es enero. Los valores están basados en el calendario académico.
*   Día de la semana: Los valores de posibilidad están divididos en día laboral, sábado y domingo, siendo el domingo el día con menor frecuencia de entrada (solamente un 1%). Por otro lado, los días laborales respetan una distribución normal, con el miércoles siendo el valor medio y una desviación estándar de dos días.
*   Hora: Los valores de probabilidad de cada hora respetan una distribución normal, siendo las 14:00 el valor medio y con una desviación estándar de una hora.
*   Temperatura: Al igual que en los casos anteriores, la probabilidad de cada valor de temperatura respeta una distribución normal, siendo los 22 grados el valor medio y presentando una desviación estándar de 2 grados. Para lograr esto, utiliza esos valores de distribución para determinar el número de percentil del valor de temperatura obtenido con el DataFrame de temperatura para el día y hora generados en pasos anteriores, y realiza el siguiente cálculo:
Probabilidad = 1 - (abs(Número de percentil - 0.5) * 2)


## Tiempo de permanencia
Para la generación de permanencia de un vehículo en el estacionamiento, se selecciona aleatoriamente un valor de un rango entre 15 y 465 minutos, lo cual equivale a un rango de 15 minutos a 7:45 horas.
```python
def generate_stay_time():
    return random.randint(15, 465)
```

## Generación

Una vez creadas las funciones se procede a generar lo que corresponde al DataFrame en sí. Para esto se considera un promedio de entrada de 100000 vehículos por año, lo que corresponde a un total de 465000 vehículos entrantes en el periodo 26/03/2019 a 31/10/2023.

```python
# Genera una lista de patentes
num_cars = 465000
cars = [generate_license_plate() for _ in range(num_cars)]

# Genera registros de entrada y salida para cada auto, con su respectivo tiempo de espera
data = []
for car in cars:
    entrance_time = generate_datetime()
    stay_time = generate_stay_time()
    exit_time = datetime.strptime(entrance_time, "%d/%m/%Y %H:%M:%S") + timedelta(minutes=stay_time)
    exit_time = exit_time.strftime("%d/%m/%Y %H:%M:%S")

    data.append({'License Plate': car, 'Datetime': entrance_time, 'Action': 'Entrance'})
    data.append({'License Plate': car, 'Datetime': exit_time, 'Action': 'Exit'})

# Convierte los datos a un DataFrame
df = pd.DataFrame(data)

# Convierte la columna 'Datetime' en un objeto datetime
df['Datetime'] = pd.to_datetime(df['Datetime'], format="%d/%m/%Y %H:%M:%S")

# Ordena el DataFrame por la columna 'Datetime'
df.sort_values('Datetime', ignore_index=True, inplace=True)

# Agrega la columna 'Weekday' para conocer el día de la semana
df['Weekday'] = df['Datetime'].dt.day_name()

df
```
![[Pasted image 20231211211343.png]]Una vez que ya tenemos el DataFrame con información de la entrada y salida de vehículos, procedemos a unirlo con el DataFrame de temperatura generado anteriormente. Para esto se toma el valor de temperatura conocido más cercano a la hora de entrada del vehículo.
```python
# Convierte la columna 'Datetime' en un objeto datetime
df_temp['Datetime'] = pd.to_datetime(df_temp['Datetime'], format="%d/%m/%Y %H:%M:%S")

# Redondeamos la hora de entrada del vehículo al intervalo de una hora más cercano
df['Rounded_Datetime'] = df['Datetime'].dt.round('H')

# Unimos los dos DataFrames basándonos en la fecha y hora redondeada
df_final = pd.merge(df, df_temp, how='left', left_on='Rounded_Datetime', right_on='Datetime')

# Renombramos las columnas si es necesario
df_final.rename(columns={'Datetime_x': 'Datetime'}, inplace=True)

# Eliminamos las columnas innecesarias
df_final.drop(columns={'Rounded_Datetime', 'Datetime_y'}, inplace=True)

df_final.to_csv('parking_unsta.csv')
df_final
```

![[Pasted image 20231211211409.png]]

# Modelado
Se generará un modelo de aprendizaje automático que predecirá la cantidad de autos que hay en el estacionamiento en todo momento. Para esto, se seguirá una serie de pasos, entre los que se incluye la determinación del algoritmo más apropiado para la generación de nuestro modelo, con su correspondiente optimización de hiperparámetros.
Una vez generado el modelo, utilizaremos el método de Montecarlo para generar un valor promedio y reducir la aleatoriedad lo máximo posible.

## Procesamiento de datos
Antes de crear el modelo, se deben preparar los datos ya existentes. En este caso, son dos columnas las que se deben modificar:
* Action: Se transforman los valores 'Entrance' y 'Exit' en 1 y -1, respectivamente. Esto nos permitirá luego crear una columna que lleve registro de la cantidad de vehículos que se encuentran en el estacionamiento en todo momento, simplemente sumando el valor de Action a una variable global.
* Weekdays: Se transforman los valores de esta columna en dummies, lo que permitirá una mejora en las predicciones del modelo.

```python
# Convierte la columna 'Accion' en númerica
df['Action'] = df['Action'].replace({'Entrance': 1, 'Exit': -1})

# Para la columna 'Weekday', transformamos los valores en dummies
df = pd.get_dummies(df, columns=['Weekday'])
```

### Feature Engineering
Una vez que ya están procesadas las columnas existentes, se procede a utilizarlas para crear nuevas columnas que permitan mejorar o incluso dar lugar a las predicciones del modelo a crear. Esta etapa consiste de dos acciones:
*   Crear la columna 'NumCars' que lleve registro de la cantidad de autos en el estacionamiento, utilizando la columna 'Action'
*   Dividir la columna 'Datetime' en cuatro columnas para extraer la mayor cantidad de información útil

```python
# Crea la columna 'NumCars' that keeps track of the number of cars in the parking lot
df['NumCars'] = df['Action'].cumsum()

# Divide la columna 'Datetime'
df['Hour'] = df['Datetime'].dt.hour
df['Month'] = df['Datetime'].dt.month
df['Day'] = df['Datetime'].dt.day
df['Year'] = df['Datetime'].dt.year
```

## Exploratory Data Analysis (EDA)

Antes de seleccionar el algoritmo que se vaya a utilizar para generar el modelo de aprendizaje automático, se hará un análisis exploratorio de datos para obtener información que permitirá determinar ciertos parámetros iniciales en el apartado siguiente.

### Autos en el tiempo
![[Pasted image 20231211211920.png]]

### Distribución de número de autos
![[Pasted image 20231211211938.png]]

### Número de autos por mes del año
![[Pasted image 20231211212109.png]]
![[Pasted image 20231211212118.png]]

### Relación entre el número de autos y la temperatura
![[Pasted image 20231211212151.png]]
### Relación entre el número de autos y la hora del día
![[Pasted image 20231211212200.png]]

## Selección del modelo
Con la información obtenida en el paso anterior, se procede a evaluar diferentes algoritmos para encontrar aquel que menor error absoluto medio presente, valor que se utilizará como medio de evaluación para cada algoritmo.
<br>En este caso fueron seleccionados 6 algoritmos:
1.   sklearn.linear_model.LinearRegression
2.   sklearn.ensamble.RandomForestRegressor
3.   sklearn.svm.SVR
4.   statsmodels.tsa.arima.model.ARIMA
5.   statsmodels.tsa.statespace.sarimax.SARIMAX (exog=False)
6.   statsmodels.tsa.statespace.sarimax.SARIMAX (exog=True)
<br>Los primeros tres algoritmos son algoritmos estándares de regresión lineal, mientras que los últimos tres son algoritmos auto regresivos integrados de medias móviles. Estos últimos suelen adaptarse mejor a modelos que utilizan el tiempo como variable.


```
----------
Linear Regression MAE: 3.7362557409863304
Random Forest MAE: 1.3089425513819792
Support Vector Regression MAE: 4.018157853677995
ARIMA MAE: 4.152170783914354
SARIMA MAE: 6.155653502520917
SARIMAX MAE: 3.7362174857851707
----------
```

Se puede apreciar que el modelo con el menor valor de error absoluto medio fue el de Random Forest Regressor. Esto encuentra una explicación en el hecho de que, a pesar de que se trabaja con una variable temporal en el modelo, el valor medio de los datos responde siempre a los mismos valores numéricos, sin necesidad de tomarlos como fechas.
Por otro lado, cabe destacar que los resultado obtenido con el DataFrame simulado puede no concordar con los resultados obtenidos con un DataFrame con datos reales. Sin embargo, a fines de realizar un análisis estimatorio, se continuará trabajando con los resultados obtenidos.
