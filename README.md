# MEU CONTROLE FINANCEIRO
# Projeto de Sistemas de Informação
# Sistema simples para controlar receitas e despesas

import json
from datetime import datetime

ARQUIVO = "dados.json"

# Tupla com as categorias que podem ser usadas
categorias = ("Alimentação", "Moradia", "Transporte", "Saúde",
              "Educação", "Lazer", "Salário", "Outros")


# Carrega os dados que já foram salvos
def carregar_dados():
    try:
        arquivo = open(ARQUIVO, "r", encoding="utf-8")
        movimentacoes = json.load(arquivo)
        arquivo.close()

        # Converte os valores antigos que estavam salvos como texto
        # para número, para funcionar com esta versão do programa.
        for mov in movimentacoes:
            mov["valor"] = float(mov["valor"])

        return movimentacoes
    except FileNotFoundError:
        return []
    except json.JSONDecodeError:
        print("Não foi possível ler os dados salvos.")
        return []


# Salva as movimentações no arquivo
def salvar_dados(movimentacoes):
    arquivo = open(ARQUIVO, "w", encoding="utf-8")
    json.dump(movimentacoes, arquivo, indent=4, ensure_ascii=False)
    arquivo.close()


# Lê uma descrição e não permite que fique vazia
def ler_descricao():
    while True:
        descricao = input("Descrição: ").strip()

        if descricao == "":
            print("Digite uma descrição.")
        else:
            return descricao


# Lê um valor e verifica se é válido
def ler_valor():
    while True:
        texto = input("Valor: R$ ").replace(",", ".")

        try:
            valor = float(texto)

            if valor <= 0:
                print("O valor precisa ser maior que zero.")
            else:
                return round(valor, 2)

        except ValueError:
            print("Digite um valor numérico válido.")


# Lê uma data e verifica se ela existe
def ler_data():
    while True:
        data = input("Data (DD/MM/AAAA): ")

        try:
            datetime.strptime(data, "%d/%m/%Y")
            return data
        except ValueError:
            print("Data inválida. Tente novamente.")


# Mostra as categorias e recebe a escolha
def escolher_categoria():
    print("\nCategorias:")

    for i in range(len(categorias)):
        print(str(i + 1) + " - " + categorias[i])

    while True:
        try:
            opcao = int(input("Escolha uma categoria: "))

            if opcao >= 1 and opcao <= len(categorias):
                return categorias[opcao - 1]
            else:
                print("Categoria inválida.")

        except ValueError:
            print("Digite o número da categoria.")


# Escolhe se a despesa está paga ou pendente
def escolher_status():
    while True:
        print("1 - Paga")
        print("2 - Pendente")

        try:
            opcao = int(input("Escolha o status: "))

            if opcao == 1:
                return "Paga"
            elif opcao == 2:
                return "Pendente"
            else:
                print("Opção inválida.")

        except ValueError:
            print("Digite 1 ou 2.")


# Gera um número para identificar cada movimentação
def gerar_id(movimentacoes):
    maior_id = 0

    for mov in movimentacoes:
        if mov["id"] > maior_id:
            maior_id = mov["id"]

    return maior_id + 1


# Cadastra uma receita ou uma despesa
def cadastrar_movimentacao(movimentacoes, tipo):
    print("\n--- CADASTRAR", tipo.upper(), "---")

    descricao = ler_descricao()
    valor = ler_valor()
    data = ler_data()
    categoria = escolher_categoria()

    if tipo == "despesa":
        status = escolher_status()
    else:
        status = "-"

    nova_movimentacao = {
        "id": gerar_id(movimentacoes),
        "tipo": tipo,
        "descricao": descricao,
        "valor": valor,
        "data": data,
        "categoria": categoria,
        "status": status
    }

    movimentacoes.append(nova_movimentacao)
    salvar_dados(movimentacoes)

    print("Movimentação cadastrada com sucesso!")


# Mostra todas as movimentações cadastradas
def consultar_movimentacoes(movimentacoes):
    print("\n--- MOVIMENTAÇÕES CADASTRADAS ---")

    if len(movimentacoes) == 0:
        print("Nenhuma movimentação cadastrada.")
    else:
        for mov in movimentacoes:
            print("--------------------------------")
            print("ID:", mov["id"])
            print("Tipo:", mov["tipo"])
            print("Descrição:", mov["descricao"])
            print("Valor: R$ %.2f" % mov["valor"])
            print("Data:", mov["data"])
            print("Categoria:", mov["categoria"])
            print("Status:", mov["status"])


# Calcula e mostra o saldo
def consultar_saldo(movimentacoes):
    receitas = 0
    despesas = 0
    pendentes = 0

    for mov in movimentacoes:
        if mov["tipo"] == "receita":
            receitas = receitas + mov["valor"]
        elif mov["tipo"] == "despesa":
            despesas = despesas + mov["valor"]

            if mov["status"] == "Pendente":
                pendentes = pendentes + mov["valor"]

    saldo = receitas - despesas

    print("\n--- SALDO FINANCEIRO ---")
    print("Total de receitas: R$ %.2f" % receitas)
    print("Total de despesas: R$ %.2f" % despesas)
    print("Despesas pendentes: R$ %.2f" % pendentes)
    print("Saldo: R$ %.2f" % saldo)


# Procura uma movimentação pelo número de identificação
def buscar_movimentacao(movimentacoes, id_procurado):
    for mov in movimentacoes:
        if mov["id"] == id_procurado:
            return mov

    return None


# Edita uma movimentação cadastrada
def editar_movimentacao(movimentacoes):
    print("\n--- EDITAR MOVIMENTAÇÃO ---")

    if len(movimentacoes) == 0:
        print("Não existem movimentações para editar.")
        return

    try:
        id_procurado = int(input("Digite o ID: "))
    except ValueError:
        print("Digite um ID válido.")
        return

    mov = buscar_movimentacao(movimentacoes, id_procurado)

    if mov == None:
        print("Movimentação não encontrada.")
        return

    print("1 - Alterar descrição")
    print("2 - Alterar valor")
    print("3 - Alterar data")
    print("4 - Alterar categoria")

    if mov["tipo"] == "despesa":
        print("5 - Alterar status")

    opcao = input("Escolha o que deseja alterar: ")

    if opcao == "1":
        mov["descricao"] = ler_descricao()
    elif opcao == "2":
        mov["valor"] = ler_valor()
    elif opcao == "3":
        mov["data"] = ler_data()
    elif opcao == "4":
        mov["categoria"] = escolher_categoria()
    elif opcao == "5" and mov["tipo"] == "despesa":
        mov["status"] = escolher_status()
    else:
        print("Opção inválida.")
        return

    salvar_dados(movimentacoes)
    print("Movimentação alterada com sucesso!")


# Exclui uma movimentação
def excluir_movimentacao(movimentacoes):
    print("\n--- EXCLUIR MOVIMENTAÇÃO ---")

    if len(movimentacoes) == 0:
        print("Não existem movimentações para excluir.")
        return

    try:
        id_procurado = int(input("Digite o ID: "))
    except ValueError:
        print("Digite um ID válido.")
        return

    mov = buscar_movimentacao(movimentacoes, id_procurado)

    if mov == None:
        print("Movimentação não encontrada.")
    else:
        print("Descrição:", mov["descricao"])
        resposta = input("Deseja excluir? (s/n): ").lower()

        if resposta == "s":
            movimentacoes.remove(mov)
            salvar_dados(movimentacoes)
            print("Movimentação excluída.")
        else:
            print("Exclusão cancelada.")


# Mostra o total de despesas separado por categoria
def consultar_categorias(movimentacoes):
    print("\n--- GASTOS POR CATEGORIA ---")

    totais = {}

    for mov in movimentacoes:
        if mov["tipo"] == "despesa":
            categoria = mov["categoria"]

            if categoria in totais:
                totais[categoria] = totais[categoria] + mov["valor"]
            else:
                totais[categoria] = mov["valor"]

    if len(totais) == 0:
        print("Nenhuma despesa cadastrada.")
    else:
        for categoria in totais:
            print(categoria, ": R$ %.2f" % totais[categoria])


# Gera um relatório de acordo com o mês e o ano informados
def relatorio_mensal(movimentacoes):
    print("\n--- RELATÓRIO MENSAL ---")

    try:
        mes = int(input("Digite o mês (1 a 12): "))
        ano = int(input("Digite o ano: "))

        if mes < 1 or mes > 12:
            print("Mês inválido.")
            return

    except ValueError:
        print("Digite números válidos.")
        return

    receitas = 0
    despesas = 0
    encontrou = False

    for mov in movimentacoes:
        partes = mov["data"].split("/")
        mes_mov = int(partes[1])
        ano_mov = int(partes[2])

        if mes_mov == mes and ano_mov == ano:
            encontrou = True

            if mov["tipo"] == "receita":
                receitas = receitas + mov["valor"]
            else:
                despesas = despesas + mov["valor"]

    if encontrou == False:
        print("Não existem movimentações nesse período.")
    else:
        print("Receitas: R$ %.2f" % receitas)
        print("Despesas: R$ %.2f" % despesas)
        print("Saldo do mês: R$ %.2f" % (receitas - despesas))


# Mostra as opções do sistema
def mostrar_menu():
    print("\n==============================")
    print("     MEU CONTROLE FINANCEIRO")
    print("==============================")
    print("1 - Cadastrar receita")
    print("2 - Cadastrar despesa")
    print("3 - Consultar movimentações")
    print("4 - Consultar saldo")
    print("5 - Editar movimentação")
    print("6 - Excluir movimentação")
    print("7 - Consultar gastos por categoria")
    print("8 - Gerar relatório mensal")
    print("0 - Sair")


# Função principal do programa
def main():
    movimentacoes = carregar_dados()
    opcao = ""

    while opcao != "0":
        mostrar_menu()
        opcao = input("Escolha uma opção: ")

        if opcao == "1":
            cadastrar_movimentacao(movimentacoes, "receita")
        elif opcao == "2":
            cadastrar_movimentacao(movimentacoes, "despesa")
        elif opcao == "3":
            consultar_movimentacoes(movimentacoes)
        elif opcao == "4":
            consultar_saldo(movimentacoes)
        elif opcao == "5":
            editar_movimentacao(movimentacoes)
        elif opcao == "6":
            excluir_movimentacao(movimentacoes)
        elif opcao == "7":
            consultar_categorias(movimentacoes)
        elif opcao == "8":
            relatorio_mensal(movimentacoes)
        elif opcao == "0":
            print("Sistema encerrado. Até mais!")
        else:
            print("Opção inválida. Tente novamente.")


# Inicia o programa
main()
