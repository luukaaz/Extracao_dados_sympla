#Importação de Bibliotecas
import requests
import pandas as pd
from datetime import timezone, datetime, timedelta, date
import numpy as np
import matplotlib as plt
import json
import concurrent
from concurrent import futures


#Definição de Parametros a partir da data do dia 01/01/2024, ordenação do maior para o menor.
params = {'sort': 'DESC', 'page_size': 200, 'from': '2024-01-01', 'has_next': True}

  
#Conexão com API do Sympla a partir da biblioteca request.
url = {'s_token':'SEU_TOKEN'}
response = requests.get('https://api.sympla.com.br/public/v3/events', headers=url, params = params)
eventos = response.json()

  
#Transformação dos dados no formato de dicionário para um dataframe utilizando a biblioteca Pandas.
df = pd.DataFrame.from_dict(eventos, orient = 'index').T

  
#Loop para extrair os nomes dos eventosa partir da criação do input dos dados em listas.
lista_de_eventos = []
lista_de_nomes_eventos = []
lista_de_data_eventos = []
for i in range(len(df['data'])):
    lista_de_eventos.append(df['data'][i]['id'])
    lista_de_nomes_eventos.append(df['data'][i]['name'])
    lista_de_data_eventos.append(df['data'][i]['start_date'])
    print(df['data'][i]['name'])


#Loop para listar as ID's, nomes e datas dos eventos; e apresentação dos dados no formato de uma tabela.
lista_de_eventos_int = [int(i) for i in lista_de_eventos]
df_eventos = pd.DataFrame(lista_de_nomes_eventos, columns=['event_name'])
df_eventos['id_event'] = lista_de_eventos_int
df_eventos['start_date'] = lista_de_data_eventos
df_eventos.set_index('id_event', inplace = True)


#Loop para guardar os urls dos eventos em uma lista.
lista_de_url = []
for evento in lista_de_eventos_int:
    lista_de_url.append('https://api.sympla.com.br/public/v3/events' + '/' +  str(evento) + '/participants')
lista_de_url


# Loop para fazer as requisições de todos os dados de todos os eventos após o filtro inicial na requisição
params = {'sort': 'DESC', 'page_size': 200, 'from': '2024-01-01'}
url = {'s_token':'fe5eea9506c319c0561806c41bd98c8b276aa7699cefe1991e14090d411de446'}
lista = []
for numero in lista_de_eventos_int:
    url_requisicao = f'https://api.sympla.com.br/public/v3/events/{numero}/participants'
    response_pessoas = requests.get(url_requisicao, headers=url, params=params)
    participantes = response_pessoas.json()
    df_geral = pd.DataFrame.from_dict(participantes, orient = 'index').T
    df_geral['data'].drop_duplicates()
    lista.append(df_geral['data'])
    print(df_geral['data'])


#Filtro para retirar os dados com datas duplicadas
df_drop = pd.DataFrame(df_geral['data'])
df_drop = df_drop.drop_duplicates(subset = ['data'])


#Loop para adicionar as informações selecionadas do dataframe e mais importantes para o projeto em dicionários
lista_participantes_contador = {}
lista_participantes_informações = {}

for i in range(len(lista)):
    for j in range(len(lista[i])):
        if lista[i][j] is not None:
            first_name = lista[i][j]['first_name']
            last_name = lista[i][j]['last_name']
            complete_name = first_name + ' ' + last_name
            email = lista[i][j]['email']
            data_checkin = lista[i][j]['checkin']
            event_id = lista[i][j]['event_id']
            nome_evento = df_eventos.loc[event_id]['event_name']
            data_evento = df_eventos.loc[event_id]['start_date']

            if not lista[i][j]['custom_form']:
                cpf = None
            else:
                if lista[i][j]['custom_form'][0]['name'] == 'CPF':
                    cpf = lista[i][j]['custom_form'][0]['value']

            if not lista[i][j]['checkin'][0]['check_in_date']:
                check_in = 'Não'
            else:
                check_in = 'Sim'
            
            if not lista[i][j]['checkin']:
                data_checkin = None
            else:
                data_checkin = lista[i][j]['checkin'][0]['check_in_date']

            if not lista[i][j]['custom_form']:
                telefone = None
            else:
                if lista[i][j]['custom_form'][1]['name'] == 'Telefone':
                    telefone = lista[i][j]['custom_form'][1]['value']
                elif lista[i][j]['custom_form'][2]['name'] == 'Telefone':
                    telefone = lista[i][j]['custom_form'][2]['value']
                elif lista[i][j]['custom_form'][3]['name'] == 'Telefone':
                    telefone = lista[i][j]['custom_form'][3]['value']
                elif lista[i][j]['custom_form'][4]['name'] == 'Telefone':
                    telefone = lista[i][j]['custom_form'][4]['value']

            lista_participantes_contador[complete_name] = complete_name, email, check_in, data_checkin, cpf, telefone, nome_evento, data_evento


#Transformação das informações do dicionário em um dataframe e em seguida transposto para melhor adequação dos dados
df_dict = pd.DataFrame.from_dict(lista_participantes_contador, orient = 'index', columns = ['Nome', 'email', 'checkin', 'data_checkin', 'CPF', 'telefone', 'nome_evento', 'data_evento']).T
df_dict = df_dict.transpose()


#Exportação dos dados desejados no formato CSV
df_dict.to_csv('Participantes.csv', index = False)
