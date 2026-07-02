
EPSILON = 'ε'

def coletar_lista(mensagem_inicial, mensagem_continuar, mensagem_pergunta):
    lista = []
    while True:
        if not lista:
            item = input(f"{mensagem_inicial}: ").strip()
        else:
            item = input(f"{mensagem_continuar}: ").strip()

        if item == "":
            print('Entrada vazia não é permitida, digite algo.')
            continue

        if item in lista:
            print(f'O item "{item}" já foi adicionado, escolha outro.')
            continue

        lista.append(item)

        resposta = input(f"{mensagem_pergunta} (S/N): ").strip().upper()
        if resposta.lower() in ['s', 'sim']:
            break

    return lista


def coletar_alfabeto():
    print('\nALFABETO (Σ)')
    print('Exemplo: a, b, 0, 1 ...\n')
    resultado = coletar_lista(
        'Digite o primeiro símbolo do alfabeto (Σ)',
        'Digite o próximo símbolo do alfabeto (Σ)',
        'Deseja finalizar o alfabeto (Σ)?'
    )
    return resultado


def coletar_estados():
    print('\nESTADOS (Q)')
    print('Exemplo: q0, q1, q2 ...\n')
    lista_estados = coletar_lista(
        'Digite o primeiro estado (Q)',
        'Digite o próximo estado (Q)',
        'Deseja finalizar os estados (Q)?'
    )
    return lista_estados


def coletar_estado_inicial(estados):
    print('\nESTADO INICIAL (q0)')
    print(f'Estados disponíveis (Q): {estados}')
    while True:
        estado_inicial = input('Digite o estado inicial (q0): ').strip()
        if estado_inicial in estados:
            return estado_inicial
        else:
            print('O estado digitado não está no conjunto Q dado.')


def coletar_estados_finais(estados):
    print('\nESTADOS FINAIS (F)')
    print(f'Estados disponíveis (Q): {estados}')
    lista_estados = []

    while True:
        estado_final = input('Digite um estado final (F): ').strip()

        if estado_final in estados:
            if estado_final in lista_estados:
                print('Esse estado já está em F.')
            else:
                lista_estados.append(estado_final)
        else:
            print('Este estado não está contido no conjunto Q.')

        pergunta = input('Deseja adicionar mais estados finais (F)? (S/N): ').strip()

        if pergunta.lower() in ['s', 'sim']:
            continue
        else:
            break

    if not lista_estados:
        print('Aviso: nenhum estado final foi definido (F = vazio).')

    return lista_estados


def coletar_transicao(estados, alfabeto):
    print('\nFUNÇÃO DE TRANSIÇÃO (δ)')
    print(f'Estados (Q): {estados}')
    print(f'Alfabeto (Σ): {alfabeto}')
    print(f'Dica: use "{EPSILON}" (ou digite "epsilon") para transição-ε (só faz sentido em AFN).')

    transicoes = {}
    while True:
        while True:
            estado_origem = input('Digite o estado de origem (para δ): ').strip()
            if estado_origem not in estados:
                print('Estado não existe em Q.')
                continue
            break

        while True:
            simbolo_lido = input('Digite o símbolo lido (de Σ, ou ε): ').strip()
            if simbolo_lido.lower() == 'epsilon':
                simbolo_lido = EPSILON
            if simbolo_lido != EPSILON and simbolo_lido not in alfabeto:
                print('Símbolo não existe em Σ (e não é ε).')
                continue
            break

        while True:
            estado_destino = input('Digite o estado de destino (para δ): ').strip()
            if estado_destino not in estados:
                print('Estado não existe em Q.')
                continue
            break

        chave = (estado_origem, simbolo_lido)
        if chave in transicoes:
            if estado_destino not in transicoes[chave]:
                transicoes[chave].append(estado_destino)
        else:
            transicoes[chave] = [estado_destino]

        print(f'  δ({estado_origem}, {simbolo_lido}) = {transicoes[chave]}')

        pergunta = input('Deseja adicionar mais transições (δ)? (S/N): ').strip()
        if pergunta.lower() not in ['s', 'sim']:
            break

    return transicoes


def coletar_palavras():
    print('\nPALAVRAS PARA TESTAR----------')
    print('Exemplo: aba, 010, bb  (para a palavra vazia, apenas pressione ENTER)\n')
    palavras = []
    while True:
        palavra = input('Digite uma palavra para simular: ')
        palavras.append(palavra)
        pergunta = input('Deseja adicionar mais palavras? (S/N): ').strip()
        if pergunta.lower() not in ['s', 'sim']:
            break
    return palavras

def detectar_tipo(transicoes):
    """
    Um autômato é AFN se: existe qualquer transição-ε, OU existe algum par (estado, símbolo) com mais de um destino possível.
    Caso contrário, é um AFD.
    """
    for chave, destinos in transicoes.items():
        if chave[1] == EPSILON:
            return 'AFN'
        if len(destinos) > 1:
            return 'AFN'
    return 'AFD'

def fecho_epsilon(estados, transicoes):
    fechado = set(estados)
    pilha = list(estados)

    while pilha:
        estado_atual = pilha.pop()
        destinos_epsilon = transicoes.get((estado_atual, EPSILON), [])
        for destino in destinos_epsilon:
            if destino not in fechado:
                fechado.add(destino)
                pilha.append(destino)

    return fechado


def simular_palavra(transicoes, estado_inicial, estados_finais, palavra):
    estados_atuais = fecho_epsilon({estado_inicial}, transicoes)

    for simbolo in palavra:
        proximos_estados = set()
        for estado in estados_atuais:
            proximos_estados |= set(transicoes.get((estado, simbolo), []))

        estados_atuais = fecho_epsilon(proximos_estados, transicoes)

        if not estados_atuais:
            return False

    return len(estados_atuais & set(estados_finais)) > 0


def executar_simulacao(resultado):
    print('\n SIMULAÇÃO DE PALAVRAS ')
    transicoes = resultado['Funções de transição (δ): ']
    inicial = resultado['Estado inicial (q0): ']
    finais = resultado['Estados finais (F): ']

    palavras = coletar_palavras()

    print('\n RESULTADOS ')
    for palavra in palavras:
        aceita = simular_palavra(transicoes, inicial, finais, palavra)
        rotulo_palavra = palavra if palavra != '' else 'ε (palavra vazia)'
        status = 'ACEITA' if aceita else 'REJEITADA'
        print(f'Palavra "{rotulo_palavra}": {status}')

def entrada_automato():
    print(' SIMULADOR DE AUTÔMATO FINITO (AFD / AFN)')
    print(' Você vai definir: Q, Σ, δ, q0 e F')

    alfabeto = coletar_alfabeto()
    estados = coletar_estados()
    inicial = coletar_estado_inicial(estados)
    final = coletar_estados_finais(estados)
    transicoes = coletar_transicao(estados, alfabeto)
    tipo = detectar_tipo(transicoes)

    resultado = {
        'Alfabeto (Σ): ': alfabeto,
        'Estados (Q): ': estados,
        'Estado inicial (q0): ': inicial,
        'Estados finais (F): ': final,
        'Funções de transição (δ): ': transicoes,
        'Tipo do autômato: ': tipo
    }

    return resultado


if __name__ == '__main__':
    resultado = entrada_automato()

    print('\n AUTÔMATO DEFINIDO')
    for chave, valor in resultado.items():
        print(f'{chave}{valor}')

    executar_simulacao(resultado)