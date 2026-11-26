# monitor-motores-python
Prova prática

## Funcionamento do Monitoramento de Motores Industriais

Primeiro temos os limites dos __motores__.

Parâmetro | Limite | Unidade
:--- | :---: | ---:
Temperatura Max | 80.0 | C
Corrente Max | 18.5 | A
Vibração Max | 4.5 | mm/s
RPM Min | 1400 | --
RPM Max | 1800 | --

### Depois ele vai fazer a leitura do __motor__.

````
def validar_leitura(motor):
    campos_obrigatorios = ["id", "setor", "temperatura", "corrente", "vibracao", "rpm", "potencia_kw"]
    for campo in campos_obrigatorios:
        if campo not in motor or motor[campo] is None:
            return False
    if motor["temperatura"] < 0 or motor["corrente"] < 0 or motor["vibracao"] < 0 or motor["rpm"] < 0:
        return False
    return True


def obter_leituras_validas(lista_motores):
    return [m for m in lista_motores if validar_leitura(m)]
````

#### E _valida-la_

### Após isso o sistema irá fazer a analise de _alertas e criticidade_ onde o sistema vai comparar a leitura atual do __motor__ com os limites predefinidos.

````
def analisar_alertas(motor):
    alertas = []
    if motor["temperatura"] > LIMITES["temperatura_max"]:
        alertas.append(f"Superaquecimento ({motor['temperatura']:.1f} C)")
    if motor["corrente"] > LIMITES["corrente_max"]:
        alertas.append(f"Sobrecorrente ({motor['corrente']:.1f} A)")
    if motor["vibracao"] > LIMITES["vibracao_max"]:
        alertas.append(f"Vibracao elevada ({motor['vibracao']:.1f} mm/s)")
    if motor["rpm"] < LIMITES["rpm_min"] or motor["rpm"] > LIMITES["rpm_max"]:
        alertas.append(f"Rotacao fora da faixa ({motor['rpm']} RPM)")

    if len(alertas) == 0:
        status = "NORMAL"
    elif len(alertas) == 1:
        status = "ATENCAO"
    else:
        status = "CRITICO"

    return status, alertas
````

### O __sistema__ também faz um cálculo das estatísticas operacionais, basicamente tirando uma média dos dados obtidos.

````
def calcular_estatisticas(motores):
    total = len(motores)
    if total == 0:
        return {}

    temp_media = sum(m["temperatura"] for m in motores) / total
    corrente_media = sum(m["corrente"] for m in motores) / total
    potencia_total = sum(m["potencia_kw"] for m in motores)
    maior_temp = max(motores, key=lambda m: m["temperatura"])

    return {
        "total_motores": total,
        "temperatura_media": round(temp_media, 2),
        "corrente_media": round(corrente_media, 2),
        "potencia_total_kw": round(potencia_total, 2),
        "motor_mais_quente": maior_temp["id"],
    }
````
### E por fim tem o _painel de exibição_ e execução principal que vai exibir tudo que nós precisamos saber sobre o que está acontecendo com os __motores__.