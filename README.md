#alunos:
josé enthonny oliveira silva / 01988470
Thiago Henrique Pereira / 01892849
Kauãn Felipe da Silva Melo / 01892125
Lucas Silva Conceição / 01887741 
Daniel Vinicio / 01886913 #




import psycopg2

# Configurações de acesso ao PostgreSQL (ajuste conforme o seu pgAdmin / servidor)
DB_CONFIG = {
    "dbname": "TRABALHO_MAFRA",
    "user": "postgres",
    "password": "senha",
    "host": "localhost",
    "port": "5432",
}


def conectar_banco():
  """Realiza a conexão com o banco de dados PostgreSQL."""
  conexao = psycopg2.connect(**DB_CONFIG)
  return conexao


def criar_tabela():
  """Cria automaticamente a tabela de alunos se ela não existir."""
  conexao = conectar_banco()
  cursor = conexao.cursor()
  cursor.execute("""
        CREATE TABLE IF NOT EXISTS alunos (
            id SERIAL PRIMARY KEY,
            nome VARCHAR(100) NOT NULL,
            idade INTEGER NOT NULL,
            nota NUMERIC(4,2) NOT NULL
        )
    """)
  conexao.commit()
  cursor.close()
  conexao.close()


def cadastrar_aluno():
  print("\n--- CADASTRAR ALUNO ---")
  nome = input("Digite o nome do aluno: ").strip()
  if not nome:
    print("Erro: O nome não pode estar vazio!")
    return

  try:
    idade = int(input("Digite a idade do aluno: "))
    if idade <= 0:
      print("Erro: A idade deve ser maior que zero!")
      return
  except ValueError:
    print("Erro: Digite um número inteiro válido para a idade!")
    return

  try:
    nota = float(input("Digite a nota do aluno (0 a 10): "))
    if not (0 <= nota <= 10):
      print("Erro: A nota deve estar entre 0 e 10!")
      return
  except ValueError:
    print("Erro: Digite um número válido para a nota!")
    return

  conexao = conectar_banco()
  cursor = conexao.cursor()
  # Consulta parametrizada com %s para segurança contra SQL Injection
  cursor.execute(
      "INSERT INTO alunos (nome, idade, nota) VALUES (%s, %s, %s)",
      (nome, idade, nota),
  )
  conexao.commit()
  cursor.close()
  conexao.close()
  print(f"Sucesso: Aluno {nome} cadastrado com sucesso!")


def listar_alunos():
  print("\n--- LISTA DE TODOS OS ALUNOS ---")
  conexao = conectar_banco()
  cursor = conexao.cursor()
  cursor.execute("SELECT id, nome, idade, nota FROM alunos ORDER BY id")
  alunos = cursor.fetchall()
  cursor.close()
  conexao.close()

  if not alunos:
    print("Nenhum aluno cadastrado no sistema.")
    return

  for aluno in alunos:
    print(
        f"ID: {aluno[0]} | Nome: {aluno[1]} | Idade: {aluno[2]} | Nota:"
        f" {float(aluno[3]):.2f}"
    )


def buscar_aluno_por_nome():
  print("\n--- BUSCAR ALUNO POR NOME ---")
  termo = input("Digite o nome (ou parte do nome) que deseja buscar: ").strip()
  if not termo:
    print("Erro: O campo de busca não pode estar vazio.")
    return

  conexao = conectar_banco()
  cursor = conexao.cursor()
  # ILIKE busca ignorando diferenças entre maiúsculas e minúsculas no Postgres
  cursor.execute(
      "SELECT id, nome, idade, nota FROM alunos WHERE nome ILIKE %s",
      (f"%{termo}%",),
  )
  alunos = cursor.fetchall()
  cursor.close()
  conexao.close()

  if not alunos:
    print("Nenhum aluno encontrado com esse nome.")
    return

  print("\nResultados encontrados:")
  for aluno in alunos:
    print(
        f"ID: {aluno[0]} | Nome: {aluno[1]} | Idade: {aluno[2]} | Nota:"
        f" {float(aluno[3]):.2f}"
    )


def atualizar_aluno():
  print("\n--- ATUALIZAR DADOS DE UM ALUNO ---")
  try:
    id_aluno = int(input("Digite o ID do aluno que deseja atualizar: "))
  except ValueError:
    print("Erro: ID inválido.")
    return

  conexao = conectar_banco()
  cursor = conexao.cursor()
  cursor.execute(
      "SELECT id, nome, idade, nota FROM alunos WHERE id = %s", (id_aluno,)
  )
  aluno = cursor.fetchone()

  if not aluno:
    print(f"Erro: Não existe aluno cadastrado com o ID {id_aluno}.")
    cursor.close()
    conexao.close()
    return

  print(
      f"Aluno atual -> Nome: {aluno[1]} | Idade: {aluno[2]} | Nota:"
      f" {float(aluno[3]):.2f}"
  )

  novo_nome = input(
      "Digite o novo nome (ou aperte Enter para manter): "
  ).strip()
  novo_nome = novo_nome if novo_nome else aluno[1]

  try:
    entrada_idade = input(
        "Digite a nova idade (ou aperte Enter para manter): "
    ).strip()
    nova_idade = int(entrada_idade) if entrada_idade else aluno[2]
  except ValueError:
    print("Erro: Idade inválida. Operação cancelada.")
    cursor.close()
    conexao.close()
    return

  try:
    entrada_nota = input(
        "Digite a nova nota (ou aperte Enter para manter): "
    ).strip()
    nova_nota = float(entrada_nota) if entrada_nota else float(aluno[3])
  except ValueError:
    print("Erro: Nota inválida. Operação cancelada.")
    cursor.close()
    conexao.close()
    return

  cursor.execute(
      "UPDATE alunos SET nome = %s, idade = %s, nota = %s WHERE id = %s",
      (novo_nome, nova_idade, nova_nota, id_aluno),
  )
  conexao.commit()
  cursor.close()
  conexao.close()
  print("Sucesso: Dados do aluno atualizados com sucesso!")


def remover_aluno():
  print("\n--- REMOVER ALUNO ---")
  try:
    id_aluno = int(input("Digite o ID do aluno que deseja remover: "))
  except ValueError:
    print("Erro: ID inválido.")
    return

  conexao = conectar_banco()
  cursor = conexao.cursor()
  cursor.execute("SELECT id, nome FROM alunos WHERE id = %s", (id_aluno,))
  aluno = cursor.fetchone()

  if not aluno:
    print(f"Erro: Aluno com ID {id_aluno} não foi encontrado.")
    cursor.close()
    conexao.close()
    return

  cursor.execute("DELETE FROM alunos WHERE id = %s", (id_aluno,))
  conexao.commit()
  cursor.close()
  conexao.close()
  print(f"Sucesso: Aluno '{aluno[1]}' removido com sucesso!")


def mostrar_media_geral():
  print("\n--- MÉDIA GERAL DAS NOTAS ---")
  conexao = conectar_banco()
  cursor = conexao.cursor()
  # Utilizando a função de agregação AVG() do SQL
  cursor.execute("SELECT AVG(nota) FROM alunos")
  resultado = cursor.fetchone()
  cursor.close()
  conexao.close()

  if resultado and resultado[0] is not None:
    media = float(resultado[0])
    print(f"A média geral das notas dos alunos cadastrados é: {media:.2f}")
  else:
    print("Não há alunos cadastrados para calcular a média.")


def menu():
  try:
    criar_tabela()
  except Exception as e:
    print(
        f"Erro ao conectar ao PostgreSQL. Verifique suas credenciais no"
        f" código: {e}"
    )
    return

  while True:
    print("\n=================================")
    print("   SISTEMA DE CADASTRO - POSTGRES")
    print("=================================")
    print("1. Cadastrar aluno")
    print("2. Listar todos os alunos")
    print("3. Buscar aluno pelo nome")
    print("4. Atualizar dados de um aluno")
    print("5. Remover aluno")
    print("6. Mostrar média geral das notas")
    print("7. Sair")

    opcao = input("Escolha uma opção (1-7): ").strip()

    if opcao == "1":
      cadastrar_aluno()
    elif opcao == "2":
      listar_alunos()
    elif opcao == "3":
      buscar_aluno_por_nome()
    elif opcao == "4":
      atualizar_aluno()
    elif opcao == "5":
      remover_aluno()
    elif opcao == "6":
      mostrar_media_geral()
    elif opcao == "7":
      print("\nSaindo do sistema. Até mais!")
      break
    else:
      print("Opção inválida! Escolha um número entre 1 e 7.")


if __name__ == "__main__":
  menu()
