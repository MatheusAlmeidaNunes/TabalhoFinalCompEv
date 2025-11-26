import math
import random
import copy
import time # Adicionado para medir o tempo de execução

# Dados fornecidos pelo usuário
DATABASE = [
    {
        "nome": "Posto de Coleta Betim, Betim - MG, Brasil",
        "latitude": -19.948951365167947,
        "longitude": -44.18947764737711
    },
    {
        "nome": "Posto de Coleta HJK, Belo Horizonte - MG, Brasil",
        "latitude": -19.99112970479588,
        "longitude": -43.99933184552746
    },
    {
        "nome": "Posto de Coleta do Shopping Estação, Belo Horizonte - MG, Brasil",
        "latitude": -19.820416617863067,
        "longitude": -43.947005052313436
    },
    {
        "nome": "Hemonúcleo de Ponte Nova, Ponte Nova - MG, Brasil",
        "latitude": -20.41845725060825,
        "longitude": -42.913137559012576
    },
    {
        "nome": "PACE Viçosa, Viçosa - MG, Brasil",
        "latitude": -20.753048549117512,
        "longitude": -42.884530687528034
    },
    {
        "nome": "PACE Conselheiro Lafaiete, Conselheiro Lafaiete - MG, Brasil",
        "latitude": -20.689086094323866,
        "longitude": -43.78062668229617
    },
    {
        "nome": "Hemonúcleo de Passos, Passos - MG, Brasil",
        "latitude": -20.72636608122992,
        "longitude": -46.61192437434855
    },
    {
        "nome": "PACE Itajubá, Itajubá - MG, Brasil",
        "latitude": -22.42614689617892,
        "longitude": -45.44131211294381
    },
    {
        "nome": "PACE Varginha, Varginha - MG, Brasil",
        "latitude": -22.426117143868215,
        "longitude": -45.441483774312665
    },
    {
        "nome": "PACE São Sebastião do Paraíso, São Sebastião do Paraíso - MG, Brasil",
        "latitude": -20.919704824777593,
        "longitude": -46.97995215715348
    },
    {
        "nome": "Hemocentro de Montes Claros, Montes Claros - MG, Brasil",
        "latitude": -16.731598166396648,
        "longitude": -43.87114936277737
    },
    {
        "nome": "Hemocentro de Governador Valadares, Governador Valadares - MG, Brasil",
        "latitude": -18.851338248938855,
        "longitude": -41.94704885904268
    },
    {
        "nome": "Posto de Coleta de Poços de Caldas, Poços de Caldas - MG, Brasil",
        "latitude": -21.78337433601221,
        "longitude": -46.55325551045953
    },
    {
        "nome": "Hemonúcleo de Divinópolis, Divinópolis - MG, Brasil",
        "latitude": -20.117777251198792,
        "longitude": -44.88313051853854
    },
    {
        "nome": "Hemonúcleo de Patos de Minas, Patos de Minas - MG, Brasil",
        "latitude": -18.593878824669936,
        "longitude": -46.51480371671857
    },
    {
        "nome": "Hemocentro de Pouso Alegre, Pouso Alegre - MG, Brasil",
        "latitude": -22.225901261317368,
        "longitude": -45.92938362034376
    },
    # O Hemocentro de BH (depósito) deve ser o PRIMEIRO na lista.
    # Vou usá-lo como o depósito (índice 0) para o VRP.
    {
        "nome": "Hemocentro de Belo Horizonte, Belo Horizonte - MG, Brasil",
        "latitude": -19.924461235476603,
        "longitude": -43.931593457173186
    },
    {
        "nome": "Hemocentro de Uberlândia, Uberlândia - MG, Brasil",
        "latitude": -18.880903548283282,
        "longitude": -48.260862580482055
    },
    {
        "nome": "Hemocentro de Juiz de Fora, Juiz de Fora - MG, Brasil",
        "latitude": -21.753335389549427,
        "longitude": -43.351818955287314
    }
]

# Dados do Depósito: O código assume que o depósito é o Hemocentro Coordenador de BH
DEPOSITO = {
    "nome": "Hemocentro Coordenador de BH (DEPÓSITO)",
    "latitude": -19.92444108073885,
    "longitude": -43.931850946026636
}

# Garante que o Depósito seja o PRIMEIRO ponto (índice 0) para a lógica VRP
try:
    # Remove a entrada "Hemocentro de Belo Horizonte" se ela for duplicada ou semelhante
    # A verificação é um pouco frágil, mas tenta encontrar e remover
    idx_bh = -1
    for i, p in enumerate(DATABASE):
        if "Belo Horizonte" in p["nome"] and "Hemocentro" in p["nome"]:
            idx_bh = i
            break
    if idx_bh != -1:
        DATABASE.pop(idx_bh)
except:
    pass # Ignora erros se a remoção falhar

DATABASE.insert(0, DEPOSITO)

NUM_PONTOS = len(DATABASE)
PONTOS = [(ponto["latitude"], ponto["longitude"]) for ponto in DATABASE]
NOMES = [ponto["nome"] for ponto in DATABASE]


## 1. Funções de Distância e Custo

def haversine(p1, p2):
    """Calcula a distância de Haversine (geodésica) entre dois pontos em km."""
    R = 6371  # Raio da Terra em quilômetros
    lat1, lon1 = p1
    lat2, lon2 = p2

    # Converte graus para radianos
    lat1_rad = math.radians(lat1)
    lon1_rad = math.radians(lon1)
    lat2_rad = math.radians(lat2)
    lon2_rad = math.radians(lon2)

    dlat = lat2_rad - lat1_rad
    dlon = lon2_rad - lon1_rad

    a = math.sin(dlat / 2)**2 + math.cos(lat1_rad) * math.cos(lat2_rad) * math.sin(dlon / 2)**2
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))

    return R * c

# Pré-cálculo da matriz de distância (agora em KM)
DIST_MATRIX = [
    [haversine(PONTOS[i], PONTOS[j]) for j in range(NUM_PONTOS)]
    for i in range(NUM_PONTOS)
]

def calcular_custo(rotas):
    """
    Calcula o custo total (distância em KM) da solução.
    A rota deve começar e terminar no depósito (índice 0).
    """
    custo_total = 0
    for rota in rotas:
        if not rota:
            continue
        
        # Distância Depósito -> Primeiro ponto
        custo_total += DIST_MATRIX[0][rota[0]]
        
        # Distância entre os pontos da rota
        for i in range(len(rota) - 1):
            custo_total += DIST_MATRIX[rota[i]][rota[i+1]]
            
        # Distância Último ponto -> Depósito
        custo_total += DIST_MATRIX[rota[-1]][0]
        
    return custo_total


## 2. Funções de Solução Inicial e Geração de Vizinhança

def solucao_gulosa_inicial(num_veiculos):
    """
    Gera uma solução inicial simples (distribuição equilibrada e aleatória).
    (Sugestão de melhoria futura: usar uma heurística de savings ou vizinho mais próximo).
    """
    pontos_a_visitar = list(range(1, NUM_PONTOS)) # Exclui o depósito (índice 0)
    random.shuffle(pontos_a_visitar)

    # Atribui os pontos de forma equilibrada entre os veículos
    rotas = [[] for _ in range(num_veiculos)]
    for i, ponto_idx in enumerate(pontos_a_visitar):
        rotas[i % num_veiculos].append(ponto_idx)
    
    return rotas


def custo_incremental_swap_intra(rota, i, j):
    """Calcula a mudança de custo ao trocar dois nós (i e j) na mesma rota."""
    delta = 0
    n = len(rota)
    
    if i == j or n < 2:
        return 0

    # Índices dos nós envolvidos
    idx_i, idx_j = rota[i], rota[j]
    
    # 1. Arestas Perdidas (custo antigo)
    
    # Arestas de i (se não for o início/fim)
    if i > 0: # Aresta (i-1) -> i
        delta -= DIST_MATRIX[rota[i-1]][idx_i]
    else: # Aresta Depósito -> i (0 -> i)
        delta -= DIST_MATRIX[0][idx_i]
        
    if i < n - 1 and i + 1 != j: # Aresta i -> (i+1)
        delta -= DIST_MATRIX[idx_i][rota[i+1]]
    elif i == n - 1: # Aresta i -> Depósito (i -> 0)
        delta -= DIST_MATRIX[idx_i][0]

    # Arestas de j (se não for vizinho de i)
    if j > 0 and j != i + 1: # Aresta (j-1) -> j
        delta -= DIST_MATRIX[rota[j-1]][idx_j]
    
    if j < n - 1: # Aresta j -> (j+1)
        delta -= DIST_MATRIX[idx_j][rota[j+1]]
    elif j == n - 1 and j != i: # Aresta j -> Depósito (j -> 0)
        delta -= DIST_MATRIX[idx_j][0]
        
    # Caso especial: i e j são vizinhos (i+1 = j)
    if j == i + 1:
        # Perde a aresta i -> j (idx_i -> idx_j)
        delta -= DIST_MATRIX[idx_i][idx_j]
        # Ganha a aresta j -> i (idx_j -> idx_i)
        delta += DIST_MATRIX[idx_j][idx_i]
        # As outras adjacências são tratadas acima/abaixo

    # 2. Arestas Novas (custo novo)
    
    # Adjacências de j (na posição antiga de i)
    if i > 0: # Aresta (i-1) -> j
        delta += DIST_MATRIX[rota[i-1]][idx_j]
    else: # Aresta Depósito -> j (0 -> j)
        delta += DIST_MATRIX[0][idx_j]

    if i < n - 1 and i + 1 != j: # Aresta j -> (i+1)
        delta += DIST_MATRIX[idx_j][rota[i+1]]
    elif i == n - 1: # Aresta j -> Depósito (j -> 0)
        delta += DIST_MATRIX[idx_j][0]
        
    # Adjacências de i (na posição antiga de j)
    if j > 0 and j != i + 1: # Aresta (j-1) -> i
        delta += DIST_MATRIX[rota[j-1]][idx_i]

    if j < n - 1: # Aresta i -> (j+1)
        delta += DIST_MATRIX[idx_i][rota[j+1]]
    elif j == n - 1 and j != i: # Aresta i -> Depósito (i -> 0)
        delta += DIST_MATRIX[idx_i][0]

    return delta


def buscar_melhor_vizinho(solucao_atual, custo_atual, melhor_custo_global, k, tenure, lista_tabu):
    """Busca o melhor vizinho usando custos incrementais e respeitando a Lista Tabu."""
    
    melhor_vizinho = None
    melhor_custo_vizinho = float('inf')
    melhor_movimento = None

    num_veiculos = len(solucao_atual)

    # --- 1. Movimento Intra-Rota (Swap de 2 nós) ---
    for v_idx, rota in enumerate(solucao_atual):
        n = len(rota)
        if n >= 2:
            for i in range(n):
                for j in range(i + 1, n):
                    
                    # CÁLCULO DE CUSTO INCREMENTAL
                    delta_custo = custo_incremental_swap_intra(rota, i, j)
                    custo_vizinho = custo_atual + delta_custo
                    
                    # Movimento: Troca dos índices dos pontos
                    node_i_idx, node_j_idx = rota[i], rota[j]
                    movimento = ('intra', v_idx, (node_i_idx, node_j_idx))
                    
                    # Checagem Tabu e Aspiração
                    is_tabu = any(m[0] == movimento and k < m[1] for m in lista_tabu)
                    is_aspirado = custo_vizinho < melhor_custo_global

                    if (not is_tabu or is_aspirado) and custo_vizinho < melhor_custo_vizinho:
                        # Se for o melhor válido, gera a solução vizinha (deep copy)
                        melhor_custo_vizinho = custo_vizinho
                        melhor_movimento = movimento
                        melhor_vizinho = copy.deepcopy(solucao_atual)
                        melhor_vizinho[v_idx][i], melhor_vizinho[v_idx][j] = melhor_vizinho[v_idx][j], melhor_vizinho[v_idx][i]


    # --- 2. Movimento Inter-Rota (Swap de 1 nó) ---
    # Este cálculo incremental é mais complexo, vamos usar a checagem com full copy por simplicidade inicial
    for v1_idx in range(num_veiculos):
        for v2_idx in range(v1_idx + 1, num_veiculos):
            rota1 = solucao_atual[v1_idx]
            rota2 = solucao_atual[v2_idx]
            if rota1 and rota2:
                for i in range(len(rota1)):
                    for j in range(len(rota2)):
                        vizinho = copy.deepcopy(solucao_atual)
                        
                        # Troca os nós i e j entre as rotas
                        node_i = vizinho[v1_idx].pop(i)
                        node_j = vizinho[v2_idx].pop(j)
                        vizinho[v1_idx].insert(i, node_j)
                        vizinho[v2_idx].insert(j, node_i)
                        
                        custo_vizinho = calcular_custo(vizinho) # Custo completo
                        movimento = ('inter', v1_idx, v2_idx, (node_i, node_j))
                        
                        # Checagem Tabu e Aspiração
                        is_tabu = any(m[0] == movimento and k < m[1] for m in lista_tabu)
                        is_aspirado = custo_vizinho < melhor_custo_global

                        if (not is_tabu or is_aspirado) and custo_vizinho < melhor_custo_vizinho:
                            melhor_custo_vizinho = custo_vizinho
                            melhor_movimento = movimento
                            melhor_vizinho = vizinho
                        
    return melhor_vizinho, melhor_custo_vizinho, melhor_movimento



## 3. Algoritmo de Busca Tabu

def busca_tabu_vrp(num_veiculos, max_iteracoes, tenure):
    """
    Implementa o algoritmo de Busca Tabu para o VRP (agora com Haversine e otimização).
    """
    
    # 1. Inicialização
    solucao_atual = solucao_gulosa_inicial(num_veiculos)
    custo_atual = calcular_custo(solucao_atual)
    
    melhor_solucao = copy.deepcopy(solucao_atual)
    melhor_custo = custo_atual
    
    # Lista Tabu (armazena movimentos 'proibidos')
    lista_tabu = [] # Estrutura: [movimento, iteracao_tabu_expira]

    print(f"--- Busca Tabu VRP Iniciada com {num_veiculos} veículos ---")
    print(f"Custo Inicial (KM): {custo_atual:.2f}\n")

    # 2. Loop Principal
    for k in range(max_iteracoes):
        
        # 3. Busca no Vizinhança
        melhor_vizinho, melhor_custo_vizinho, melhor_movimento = buscar_melhor_vizinho(
            solucao_atual, custo_atual, melhor_custo, k, tenure, lista_tabu
        )
        
        # Se não encontrarmos um vizinho válido (todos tabu e não aspirados)
        if melhor_vizinho is None:
            # Estratégia simples: continua (pula a iteração)
            print(f"Iteração {k+1}: Estagnação. Nenhuma melhoria Tabu/Aspirada encontrada.")
            continue 

        # 4. Atualiza Solução Atual
        solucao_atual = melhor_vizinho
        custo_atual = melhor_custo_vizinho
        
        # 5. Atualiza a Lista Tabu
        # A restrição tabu é o próprio movimento (simplificado)
        lista_tabu.append([melhor_movimento, k + tenure])
        
        # Remove movimentos expirados
        lista_tabu = [m for m in lista_tabu if m[1] > k]
        
        # 6. Atualiza a Melhor Solução Global
        if custo_atual < melhor_custo:
            melhor_custo = custo_atual
            melhor_solucao = copy.deepcopy(solucao_atual)
            print(f"Iteração {k+1}: NOVO MELHOR CUSTO GLOBAL = {melhor_custo:.2f} KM")
        elif (k + 1) % 50 == 0:
             print(f"Iteração {k+1}: Custo Atual = {custo_atual:.2f} KM")

    print(f"\n--- Busca Tabu Finalizada ---")
    return melhor_solucao, melhor_custo


## 4. Execução do Algoritmo

# --- Parâmetros de Configuração ---
NUM_VEICULOS = 10 
MAX_ITERACOES = 20 # Aumentado para melhor exploração
TABU_TENURE = 10 

start_time = time.time()

melhor_rotas_indices, melhor_custo_final = busca_tabu_vrp(
    NUM_VEICULOS, MAX_ITERACOES, TABU_TENURE
)

end_time = time.time()

# --- Impressão dos Resultados ---
print("\n" + "="*70)
print(f"Melhor Roteamento Encontrado (Total de {NUM_VEICULOS} Veículos)")
print(f"Custo Total (Distância de Haversine): {melhor_custo_final:.2f} KM")
print(f"Tempo de Execução: {end_time - start_time:.2f} segundos")
print("="*70 + "\n")

for i, rota_indices in enumerate(melhor_rotas_indices):
    rota_nomes = [NOMES[0]] # Começa do Depósito
    rota_nomes.extend([NOMES[idx] for idx in rota_indices])
    rota_nomes.append(NOMES[0]) # Volta para o Depósito
    
    # Cálculo do custo individual da rota (para exibição)
    custo_rota = 0
    if rota_indices:
        # Depósito -> Primeiro ponto
        custo_rota += DIST_MATRIX[0][rota_indices[0]]
        # Pontos intermediários
        for j in range(len(rota_indices) - 1):
            custo_rota += DIST_MATRIX[rota_indices[j]][rota_indices[j+1]]
        # Último ponto -> Depósito
        custo_rota += DIST_MATRIX[rota_indices[-1]][0]

    print(f"--- Rota do Veículo {i+1} (Total de Pontos: {len(rota_indices)}, Custo: {custo_rota:.2f} KM) ---")
    print(" -> ".join(rota_nomes) + "\n")
