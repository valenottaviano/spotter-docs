Una vez codificada las funciones necesarias para poder simular para los 7 días de la semana, la cantidad de vehículos utilizando y esperando en el sistema, nos disponemos a encontrar cuál es el tiempo máximo que nuestra app [[QReader]] puede requerir sin que se sature el mismo (\*):

Procedemos a realizar sucesivas simulaciones con distintos tiempos de latencia, para cada día de la semana. 

![[Pasted image 20231129153144.png]]

Los resultados indican que el tiempo máximo que puede demorar el usuario utilizando la aplicación es de 60 segundos.


(\*) Definimos como *saturación del sistema* al punto en el que existe un 20% de probabilidad de que existan 2 autos esperando a poder ingresar.