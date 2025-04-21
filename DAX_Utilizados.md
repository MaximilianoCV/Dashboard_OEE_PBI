# 📘 DAX_Utilizados.md - Dashboard OEE Coflex

Este documento contiene todas las medidas DAX utilizadas en el modelo Power BI conectado a SQL Server para la evaluación del OEE y KPIs relacionados.

---
DEFINE
    MEASURE 'medidas'[2.2 Eficiencia PB] = VAR FECHA = MAX('0. Calendario'[Fecha])

VAR FechaMax = EOMONTH(FECHA,-1)
VAR FechaInicio = EOMONTH(FECHA,-2)+1

VAR PB = CALCULATE([2.2 Eficiencia], DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax))

RETURN PB
    MEASURE 'medidas'[2.3 Calidad PB] = VAR FECHA = MAX('0. Calendario'[Fecha])

VAR FechaMax = EOMONTH(FECHA,-1)
VAR FechaInicio = EOMONTH(FECHA,-3)+1


VAR PB = CALCULATE([2.3 Calidad], DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax))

RETURN PB
    MEASURE 'medidas'[2.3 Calidad] = DIVIDE(SUM('1. Producción (OT)'[Producción UN])-SUM('1. Defectos (OT)'[PIEZAS]),SUM('1. Producción (OT)'[Producción UN]))
    MEASURE 'medidas'[2.3 Calidad Gauge] = DIVIDE(SUM('1. Producción (OT)'[Producción UN])-SUM('1. Defectos (OT)'[PIEZAS]),SUM('1. Producción (OT)'[Producción UN]))
    MEASURE 'medidas'[2.0 % Mejora OEE] = ([2.0 OEE]- [2.0 OEE PB])
    MEASURE 'medidas'[2.2 % Mejora Eficiencia] = DIVIDE([2.2 Eficiencia], [2.2 Eficiencia PB]) - 1
    MEASURE 'medidas'[2.3 % Mejora Calidad] = DIVIDE([2.3 Calidad Gauge], [2.3 Calidad PB]) - 1
    MEASURE 'medidas'[4.1 TIempo neto Trabajado] = SUM('1. Producción (OT)'[Duración (HR)]) - SUM('1. Producción (OT)'[Horas Paros])
    MEASURE 'medidas'[3.1 Producción PB] = VAR MAQUINA = SELECTEDVALUE('0. Catalogo Lineas'[Linea])
VAR FECHA = MAX('0. Calendario'[Fecha])

VAR FechaMax = EOMONTH(FECHA,-1)
VAR FechaInicio = EOMONTH(FECHA,-4)+1

VAR PB = CALCULATE(SUM('1. Producción (OT)'[Producción UN]), DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax))

Return PB
    MEASURE 'medidas'[2.1 Disponibilidad] = VAR TIEMPOOTAL = sum('1. Producción (OT)'[Duración (HR)]) + CALCULATE(
        SUM('1. Paros (OT)'[DURACION (HR)]),
        '1. Paros (OT)'[OT] = "OT0000000"
    )
VAR TIEMPOLABORADO = TIEMPOOTAL-sum('1. Paros (OT)'[DURACION (HR)])
VAR TIEMPOPROGRAMADO = TIEMPOOTAL-SUM('1. Paros (OT)'[PARO PROGRAMADO])


VAR FORMULA = IF(DIVIDE(TIEMPOLABORADO,TIEMPOPROGRAMADO)>1,1,DIVIDE(TIEMPOLABORADO,TIEMPOPROGRAMADO))

RETURN FORMULA
    MEASURE 'medidas'[2.0 OEE] = [2.1 Disponibilidad]*[2.2 Eficiencia]*[2.3 Calidad]
    MEASURE 'medidas'[4.0 Horas Paro Almacen] = CALCULATE(
    SUM('1. Paros (OT)'[DURACION (HR)]),
    '1. Paros (OT)'[PARO] IN {"FALTA OT", "SIN SURTIR", "INCOMPLETO"}
)
    MEASURE 'medidas'[4.0 % Paros por Almacen] = DIVIDE([4.0 Horas Paro Almacen],sum('1. Paros (OT)'[PARO NO PROGRAMADO]))
    MEASURE 'medidas'[4.0 Promedio PB] = VAR MAQUINA = SELECTEDVALUE('0. Catalogo Lineas'[Linea])
VAR FECHA = MAX('0. Calendario'[Fecha])

-- Definir el rango de fechas del periodo base (últimos 4 meses antes del mes actual)
VAR FechaMax = EOMONTH(FECHA, -1) -- Último día del mes anterior
VAR FechaInicio = EOMONTH(FECHA, -4) + 1 -- Primer día del mes hace 4 meses

-- Calcular el tiempo promedio de paros en minutos
VAR PB = 
    CALCULATE(
        AVERAGE('1. Paros (OT)'[DURACION (Min)]),
        DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax) -- Filtrar por el rango de fechas
    )

RETURN PB
    MEASURE 'medidas'[4.0 Paros PB] = VAR MAQUINA = SELECTEDVALUE('0. Catalogo Lineas'[Linea])
VAR FECHA = MAX('0. Calendario'[Fecha])

-- Definir el rango de fechas del periodo base (últimos 4 meses antes del mes actual)
VAR FechaMax = EOMONTH(FECHA, -1) -- Último día del mes anterior
VAR FechaInicio = EOMONTH(FECHA, -4) + 1 -- Primer día del mes hace 4 meses

-- Calcular el total de horas de paro en el periodo base
VAR TotalHorasParo = 
    CALCULATE(
        SUM('1. Paros (OT)'[DURACION (HR)]),
        DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax) -- Filtrar por el rango de fechas
    )

-- Calcular el número de semanas en el periodo base
VAR NumeroSemanas = 
    CALCULATE(
        DISTINCTCOUNT('0. Calendario'[Sem (DiaMes)]), 
        DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax)
    )

-- Calcular el promedio semanal de paros en horas
VAR PB = 
    DIVIDE(TotalHorasParo, NumeroSemanas, 0) -- Divide entre las semanas, evita error si es 0

RETURN PB
    MEASURE 'medidas'[4.1 Tiempo Invisible] = sum('0. Catalogo Turnos'[Tiempo Hr])-[4.1 TIempo neto Trabajado]-sum('1. Paros (OT)'[DURACION (HR)])
    MEASURE 'medidas'[4.0 UsoParoPorOT] = VAR CantidadOTUnicas = 
    DISTINCTCOUNT('1. Paros (OT)'[OT])

VAR CantidadParo = 
    COUNT('1. Paros (OT)'[PARO])

VAR Resultado = DIVIDE(CantidadParO, CantidadOTUnicas, 0)

RETURN Resultado
    MEASURE 'medidas'[4.0 PB COUNTPAROS] = VAR MAQUINA = SELECTEDVALUE('0. Catalogo Lineas'[Linea])
VAR FECHA = MAX('0. Calendario'[Fecha])

-- Definir el rango de fechas del periodo base (últimos 4 meses antes del mes actual)
VAR FechaMax = EOMONTH(FECHA, -1) -- Último día del mes anterior
VAR FechaInicio = EOMONTH(FECHA, -4) + 1 -- Primer día del mes hace 4 meses

-- Calcular el promedio de duración en minutos para el paro "Liberación"
VAR PB = 
    CALCULATE(
        [4.0 UsoParoPorOT],
        DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax)
    )

RETURN PB
    MEASURE 'medidas'[4.0 HoraParoPorOT] = VAR CantidadOTUnicas = 
    DISTINCTCOUNT('1. Paros (OT)'[OT])

VAR CantidadParo = 
    SUM('1. Paros (OT)'[DURACION (Min)])

VAR Resultado = DIVIDE(CantidadParO, CantidadOTUnicas, 0)

RETURN Resultado
    MEASURE 'medidas'[2.1 Disponibilidad PB] = VAR FECHA = MAX('0. Calendario'[Fecha])

VAR FechaMax = EOMONTH(FECHA,-1)
VAR FechaInicio = EOMONTH(FECHA,-3)+1


VAR PB = CALCULATE([2.1 Disponibilidad], DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax))

RETURN PB
    MEASURE 'medidas'[2.0 OEE PB] = VAR MAQUINA = SELECTEDVALUE('0. Catalogo Lineas'[Linea])
VAR FECHA = MAX('0. Calendario'[Fecha])

VAR FechaMax = EOMONTH(FECHA,-2)
VAR FechaInicio = EOMONTH(FECHA,-3)+1

VAR PB = CALCULATE([2.1 Disponibilidad]*[2.3 Calidad]*[2.2 Eficiencia], DATESBETWEEN('0. Calendario'[Fecha], FechaInicio, FechaMax))

return PB
    MEASURE 'medidas'[2.2 Eficiencia] = VAR Piezas = SUM('1. Producción (OT)'[Producción UN])
VAR PiezasSTD = SUM('1. Producción (OT)'[STD Piezas]) 

RETURN IF(DIVIDE(Piezas,PiezasSTD)>1,1,DIVIDE(Piezas,PiezasSTD))
    MEASURE 'medidas'[2.2 Eficiencia (sin topar)] = VAR Piezas = SUM('1. Producción (OT)'[Producción UN])
VAR PiezasSTD = SUM('1. Producción (OT)'[STD Piezas]) 



RETURN DIVIDE(Piezas,PiezasSTD)
    MEASURE 'medidas'[5.0 TC Propuesta] = VAR Prod = sum('1. Producción (OT)'[Producción UN])
Var Segundos = [4.1 TIempo neto Trabajado]*3600
Var Real = Prod * 1.08

Return Divide(Segundos,Real)
    MEASURE 'medidas'[5.0 ModelosMayor100_Total] = CALCULATE(
    DISTINCTCOUNT('1. Producción (OT)'[Modelo]),
    FILTER(
        SUMMARIZE(
            '1. Producción (OT)',
            '1. Producción (OT)'[Modelo],
            "TotalEficiencia", [2.2 Eficiencia (sin topar)]
        ),
        [TotalEficiencia] > 1  -- Eficiencia acumulada > 100%
    )
)
    MEASURE 'medidas'[3.1 Producción Acumulada] = VAR FECHAMAX = MAX('0. Calendario'[Fecha])
VAR FECHAMIN = MIN('0. Calendario'[Fecha])

VAR PRODUCCIONTON = CALCULATE(SUM('1. Producción (OT)'[Producción UN]),FILTER(ALLSELECTED('1. Producción (OT)'),'1. Producción (OT)'[FECHA]<=max('1. Producción (OT)'[FECHA])))

Var resultado = PRODUCCIONTON

RETURN resultado
    MEASURE 'medidas'[3.0 Meta vs Producción] = VAR Diferencia = DIVIDE(SUM('1. Producción (OT)'[Producción UN]),SUM('4. Pronóstico'[Unidades]))

Return Diferencia
    MEASURE 'medidas'[3.0 FC PROYECTADO] = VAR FECHAMAX = MAX('0. Calendario'[Fecha])
VAR FECHAMIN = MIN('0. Calendario'[Fecha])

VAR PRODUCCIONTON = CALCULATE(SUM(Hoja1[UNIDADES]),FILTER(ALLSELECTED(Hoja1),Hoja1[FECHA]<=max(Hoja1[FECHA])))

Var resultado = PRODUCCIONTON

RETURN resultado
    MEASURE 'medidas'[3.1 Producción (3 Meses Mov)] = VAR FECHA_CONTEXTO = MAX('0. Calendario'[Fecha])
VAR INICIO_PERIODO = EOMONTH(FECHA_CONTEXTO, -3) + 1
VAR FIN_PERIODO = FECHA_CONTEXTO

VAR ProduccionUltimos3Meses =
    CALCULATE(
        SUM('1. Producción (OT)'[Producción UN]),
        DATESBETWEEN('0. Calendario'[Fecha], INICIO_PERIODO, FIN_PERIODO)
    )

VAR DiasHabilesPeriodo =
    CALCULATE(
        COUNTROWS('0. Calendario'),
        DATESBETWEEN('0. Calendario'[Fecha], INICIO_PERIODO, FIN_PERIODO),
        '0. Calendario'[Es_Habil] = TRUE()
    )

VAR PB = DIVIDE(ProduccionUltimos3Meses, DiasHabilesPeriodo)

RETURN
PB
    MEASURE 'medidas'[3.0 Pronostico Acumulado] = VAR FECHAMAX = MAX('0. Calendario'[Fecha])
VAR FECHAMIN = MIN('0. Calendario'[Fecha])

VAR PRONOSTICO = CALCULATE(SUM('4. Pronóstico Diario'[Unidades/Dia]),FILTER(ALLSELECTED('4. Pronóstico Diario'),'4. Pronóstico Diario'[Fecha]<=max('4. Pronóstico Diario'[Fecha])))

RETURN PRONOSTICO
    MEASURE 'medidas'[3.1 Producción PB Acumulado] = VAR FECHAMAX = MAX('0. Calendario'[Fecha])
VAR FECHAMIN = MIN('0. Calendario'[Fecha])

VAR INICIO_PB = EOMONTH(FECHAMIN, -4) + 1
VAR FIN_PB = EOMONTH(FECHAMIN, -1)

VAR PRODUCCIONTON = CALCULATE(SUM('1. Producción (OT)'[Producción UN]),FILTER(ALLSELECTED('1. Producción (OT)'),'1. Producción (OT)'[FECHA]<=max('1. Producción (OT)'[FECHA])))

Var resultado = PRODUCCIONTON

RETURN resultado
    MEASURE 'medidas'[3.0 Faltante Req Diario] = MAX(
    0,
    SUM(Hoja1[UNIDADES]) - SUM('1. Producción (OT)'[Producción UN])
)
    MEASURE 'medidas'[3.1 Producción Prom Semanal (3 Meses Mov)] = VAR FECHA_CONTEXTO = MAX('0. Calendario'[Fecha])
VAR INICIO_PERIODO = EOMONTH(FECHA_CONTEXTO, -3) + 1
VAR FIN_PERIODO = FECHA_CONTEXTO

-- Producción total en los últimos 3 meses
VAR ProduccionUltimos3Meses =
    CALCULATE(
        SUM('1. Producción (OT)'[Producción UN]),
        DATESBETWEEN('0. Calendario'[Fecha], INICIO_PERIODO, FIN_PERIODO),
        '0. Calendario'[Es_Habil] = TRUE()
    )

-- Contar las semanas únicas dentro del periodo de 3 meses
VAR SemanasEnPeriodo =
    CALCULATE(
        DISTINCTCOUNT('0. Calendario'[No.Semana]),
        DATESBETWEEN('0. Calendario'[Fecha], INICIO_PERIODO, FIN_PERIODO),
        '0. Calendario'[Es_Habil] = TRUE()
    )

-- Promedio semanal
VAR PB = DIVIDE(ProduccionUltimos3Meses, SemanasEnPeriodo)

RETURN
PB
    MEASURE 'medidas'[3.0 Dif Sem%] = DIVIDE(SUM('1. Producción (OT)'[Producción UN]),[3.1 Producción Prom Semanal (3 Meses Mov)])
    MEASURE 'medidas'[3.1 Producción Prom Mensual (3 Meses Mov)] = VAR FECHA_CONTEXTO = MAX('0. Calendario'[Fecha])
VAR INICIO_PERIODO = EOMONTH(FECHA_CONTEXTO, -3) + 1
VAR FIN_PERIODO = FECHA_CONTEXTO

-- Producción total en los últimos 3 meses
VAR ProduccionUltimos3Meses =
    CALCULATE(
        SUM('1. Producción (OT)'[Producción UN]),
        DATESBETWEEN('0. Calendario'[Fecha], INICIO_PERIODO, FIN_PERIODO)
    )

-- Promedio mensual (dividir entre 3)
VAR PB = DIVIDE(ProduccionUltimos3Meses, 3)

RETURN
PB
    MEASURE 'medidas'[3.0.Dif Mes%] = DIVIDE(SUM('1. Producción (OT)'[Producción UN]),[3.1 Producción Prom Mensual (3 Meses Mov)])
    MEASURE 'medidas'[3.0 Porcentaje Cumplimiento vs SO&P] = DIVIDE(SUM('1. Producción (OT)'[Producción UN]),[3.0 FC PROYECTADO])
    MEASURE 'medidas'[2.01 OEE Rest] = 1 - [2.0 OEE]
    MEASURE 'medidas'[4.0 % Paros Ajustados] = Var tiempo = SUM('1. Paros (OT)'[DURACION (HR)])-CALCULATE(
        SUM('1. Paros (OT)'[DURACION (HR)]),
        '1. Paros (OT)'[PARO] IN {"No producido", "Sin Clasificar","FUERA DE CICLO"})

Return DIVIDE (TIEMPO,SUM('1. Paros (OT)'[DURACION (HR)]))
    MEASURE 'medidas'[4.0 % Paros No Ajustados] = Var tiempo =CALCULATE(
        SUM('1. Paros (OT)'[DURACION (HR)]),
        '1. Paros (OT)'[PARO] IN {"No producido", "Sin Clasificar","FUERA DE CICLO"})

Return DIVIDE (TIEMPO,SUM('1. Paros (OT)'[DURACION (HR)]))
    MEASURE 'medidas'[2.4 Capacidad] = VAR PROD = SUM('1. Producción (OT)'[Producción UN])
VAR OEE = [2.0 OEE]
VAR MOEE = .73

VAR CAPACIDAD =DIVIDE((MOEE*PROD),OEE)

RETURN CAPACIDAD
    MEASURE 'medidas'[2.1 % Mejora Disponibilidad] = DIVIDE([2.1 Disponibilidad], [2.1 Disponibilidad PB]) - 1
    MEASURE 'medidas'[6.0 Meta] = SWITCH(
    TRUE(),
    SELECTEDVALUE('0. Catalogo Area'[Area]) = "Inyeccion", 0.60,
    0.69
)

EVALUATE
    SUMMARIZECOLUMNS(
        "2.2 Eficiencia PB", [2.2 Eficiencia PB],
        "2.3 Calidad PB", [2.3 Calidad PB],
        "2.3 Calidad", [2.3 Calidad],
        "2.3 Calidad Gauge", [2.3 Calidad Gauge],
        "2.0 % Mejora OEE", [2.0 % Mejora OEE],
        "2.2 % Mejora Eficiencia", [2.2 % Mejora Eficiencia],
        "2.3 % Mejora Calidad", [2.3 % Mejora Calidad],
        "4.1 TIempo neto Trabajado", [4.1 TIempo neto Trabajado],
        "3.1 Producción PB", [3.1 Producción PB],
        "2.1 Disponibilidad", [2.1 Disponibilidad],
        "2.0 OEE", [2.0 OEE],
        "4.0 Horas Paro Almacen", [4.0 Horas Paro Almacen],
        "4.0 % Paros por Almacen", [4.0 % Paros por Almacen],
        "4.0 Promedio PB", [4.0 Promedio PB],
        "4.0 Paros PB", [4.0 Paros PB],
        "4.1 Tiempo Invisible", [4.1 Tiempo Invisible],
        "4.0 UsoParoPorOT", [4.0 UsoParoPorOT],
        "4.0 PB COUNTPAROS", [4.0 PB COUNTPAROS],
        "4.0 HoraParoPorOT", [4.0 HoraParoPorOT],
        "2.1 Disponibilidad PB", [2.1 Disponibilidad PB],
        "2.0 OEE PB", [2.0 OEE PB],
        "2.2 Eficiencia", [2.2 Eficiencia],
        "2.2 Eficiencia (sin topar)", [2.2 Eficiencia (sin topar)],
        "5.0 TC Propuesta", [5.0 TC Propuesta],
        "5.0 ModelosMayor100_Total", [5.0 ModelosMayor100_Total],
        "3.1 Producción Acumulada", [3.1 Producción Acumulada],
        "3.0 Meta vs Producción", [3.0 Meta vs Producción],
        "3.0 FC PROYECTADO", [3.0 FC PROYECTADO],
        "3.1 Producción (3 Meses Mov)", [3.1 Producción (3 Meses Mov)],
        "3.0 Pronostico Acumulado", [3.0 Pronostico Acumulado],
        "3.1 Producción PB Acumulado", [3.1 Producción PB Acumulado],
        "3.0 Faltante Req Diario", [3.0 Faltante Req Diario],
        "3.1 Producción Prom Semanal (3 Meses Mov)", [3.1 Producción Prom Semanal (3 Meses Mov)],
        "3.0 Dif Sem%", [3.0 Dif Sem%],
        "3.1 Producción Prom Mensual (3 Meses Mov)", [3.1 Producción Prom Mensual (3 Meses Mov)],
        "3.0.Dif Mes%", [3.0.Dif Mes%],
        "3.0 Porcentaje Cumplimiento vs SO&P", [3.0 Porcentaje Cumplimiento vs SO&P],
        "2.01 OEE Rest", [2.01 OEE Rest],
        "4.0 % Paros Ajustados", [4.0 % Paros Ajustados],
        "4.0 % Paros No Ajustados", [4.0 % Paros No Ajustados],
        "2.4 Capacidad", [2.4 Capacidad],
        "2.1 % Mejora Disponibilidad", [2.1 % Mejora Disponibilidad],
        "6.0 Meta", [6.0 Meta]
    )

---

📌 *Nota: Este archivo contiene únicamente la lógica DAX sin explicaciones narradas. Para conocer el detalle de cada medida y su objetivo funcional, consulta el archivo `docs/DAX_Explicado.md` o el manual del usuario.*

