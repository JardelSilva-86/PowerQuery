# Parâmetro para calendário de feriados nacionais

Conectar na [Anbima](https://www.anbima.com.br/feriados/fer_nacionais/2019.asp)

Criar parâmetro no power query com as seguinte informações

Nome: AnoFeriado
Descrição: Parâmetro para escolher o ano com feriados nacionais dentro do site.
Tipo: Texto - pois este parâmetro vai na URL
Valor Atual: 2019

Agora utlizar este parâmetro na tabela de Feriados que foi conectada via WEB no Power query.

Ex: Script M da conexão da tabela feriados

let
    Fonte = Web.BrowserContents("https://www.anbima.com.br/feriados/fer_nacionais/" & AnoFeriado & ".asp"),
    #"Tabela extraída de HTML" = Html.Table(Fonte, {{"Column1", "TABLE.interna > * > TR > :nth-child(1)"}, {"Column2", "TABLE.interna > * > TR > :nth-child(2)"}, {"Column3", "TABLE.interna > * > TR > :nth-child(3)"}}, [RowSelector="TABLE.interna > * > TR"]),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(#"Tabela extraída de HTML", [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",{{"Data", type date}, {"Dia da Semana", type text}, {"Feriado", type text}})
in
    #"Tipo Alterado"

Depois disso, criar uma função para deixar essa escolha automática, clicar com o botão direito na tabela Feriados e ir em criar função, escolhi o nome fxFeriados para este caso.

Em seguida exclui a tabela Feriados, pois só utilizei ela para criar a função.
Agora, devemos criar uma lista com todos os anos que precisamos ter.

Para criar esa lista, criamos uma consulta nula e no editor avançado do power query escrevemos o seguinte:

let
    Anos = {2010..2021},
    #"Convertido para Tabela" = Table.FromList(Anos, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Colunas Renomeadas" = Table.RenameColumns(#"Convertido para Tabela",{{"Column1", "Ano"}}),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Colunas Renomeadas",{{"Ano", type text}})
in
    #"Tipo Alterado"

Em seguida, criaremos uma função personalizada na aba Adicionar Coluna na tabela dFeriado. escolher a fxFeriados para buscar os feriados dos anos correspondentes.
Quando carregados dados, podemos excluir os que não precisamos. Neste caso, a coluna de Ano e o parâmetro AnoFeriado.
