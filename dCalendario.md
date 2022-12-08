# Primeira Forma

let
  inicio= Date.StartOfDay(#date(2021, 01, 01)),
  fim = DateTime.Date(Date.EndOfYear(DateTime.LocalNow())),
  stop =Duration.Days(fim - inicio),
  dCalendario = List.Generate(
  ()=>0,
  each _ <= stop,
  each _ +1,
  each Date.AddDays(inicio,_)
  ),
  #"Convertido para Tabela" = Table.FromList(dCalendario, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
  #"Colunas Renomeadas" = Table.RenameColumns(#"Convertido para Tabela",{{"Column1", "Date"}}),
  #"Tipo Alterado" = Table.TransformColumnTypes(#"Colunas Renomeadas",{{"Date", type date}}),
  #"Nome Mês" = Table.AddColumn(#"Tipo Alterado", "Mês", each Date.MonthName([Date]), type text),
  #"1ºMaiúscula" = Table.TransformColumns(#"Nome Mês",{{"Mês", Text.Proper, type text}}),
  Ano = Table.AddColumn(#"1ºMaiúscula", "Ano", each Date.Year([Date]), Int64.Type),
  #"Nome Semana" = Table.AddColumn(Ano, "Semana do Mês", each Number.ToText(Date.WeekOfMonth([Date])) &"º Semana"),
  Trimestre = Table.AddColumn(#"Nome Semana", "Trimestre", each "T"& Number.ToText(Date.QuarterOfYear([Date]))),
  #"Mês Inserido" = Table.AddColumn(Trimestre, "NºMês", each Date.Month([Date]), Int64.Type),
  #"Nº Semana" = Table.AddColumn(#"Mês Inserido", "NºSemana", each Date.WeekOfMonth([Date]), Int64.Type),
  #"Nome Dia" = Table.AddColumn(#"Nº Semana", "Nome do Dia", each Date.DayOfWeekName([Date]), type text),
  #"Dia da Semana" = Table.AddColumn(#"Nome Dia", "Dia da Semana", each if Number.From(Date.DayOfWeek([Date],Day.Sunday))=6 or Number.From(Date.DayOfWeek([Date],Day.Sunday))=0 then "D.FDS" else "D.Semana"),
  #"Dia da Semana Inserido" = Table.AddColumn(#"Dia da Semana", "Nº Dia", each Date.DayOfWeek([Date]), Int64.Type),
  #"Colunas Reordenadas" = Table.ReorderColumns(#"Dia da Semana Inserido",{"Date", "Ano", "NºMês", "Mês", "Semana do Mês", "Dia da Semana", "Nome do Dia", "Trimestre", "Nº Dia", "NºSemana"})
in
  #"Colunas Reordenadas"
  
  
# Segunda Forma

let

//Datas Padrão para gerar a tabela, você pode comentar estes 2 parametros e usar as datas da sua fato para maior dinamismo.
DataInicial = #date(2018,01,01),
DataFinal = #date(Date.Year(DateTime.LocalNow()),12,31),


Duracao = Duration.Days(DataFinal-DataInicial) + 1,

ListaDatas = List.Dates(DataInicial,Duracao,#duration(1,0,0,0)),
    #"Convertido para Tabela" = Table.FromList(ListaDatas, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Colunas Renomeadas" = Table.RenameColumns(#"Convertido para Tabela",{{"Column1", "Data"}}),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Colunas Renomeadas",{{"Data", type date}}),
    #"Ano Inserido" = Table.AddColumn(#"Tipo Alterado", "Ano", each Date.Year([Data]), Int64.Type),
    #"Mês Inserido" = Table.AddColumn(#"Ano Inserido", "Mês", each Date.Month([Data]), Int64.Type),
    #"Nome do Mês Inserido" = Table.AddColumn(#"Mês Inserido", "Nome do Mês", each Date.MonthName([Data]), type text),
    #"Dia da Semana Inserido" = Table.AddColumn(#"Nome do Mês Inserido", "Dia da Semana", each Date.DayOfWeek([Data]), Int64.Type),
    #"Nome do Dia Inserido" = Table.AddColumn(#"Dia da Semana Inserido", "Nome do Dia", each Date.DayOfWeekName([Data]), type text),
    #"Primeiros Caracteres Inseridos" = Table.AddColumn(#"Nome do Dia Inserido", "Mês Abrev", each Text.Start([Nome do Mês], 3), type text),
    #"Personalização Adicionada" = Table.AddColumn(#"Primeiros Caracteres Inseridos", "Mês/Ano", each [Mês Abrev] & "/" & Text.From([Ano]), type text),
    #"Personalização Adicionada1" = Table.AddColumn(#"Personalização Adicionada", "Mês/ano Class.", each [Ano] * 100 +[Mês], type number),
    #"Colocar Cada Palavra Em Maiúscula" = Table.TransformColumns(#"Personalização Adicionada1",{{"Nome do Mês", Text.Proper, type text}, {"Nome do Dia", Text.Proper, type text}, {"Mês Abrev", Text.Proper, type text}, {"Mês/Ano", Text.Proper, type text}}),
    #"Personalização Adicionada2" = Table.AddColumn(#"Colocar Cada Palavra Em Maiúscula", "Dia Útil", each 

//Feriados Fixos

//Verifica Sabado e Domingo
if [Dia da Semana] = 0 then "NÃO" else
if [Dia da Semana] = 6 then "NÃO" else
//Verifica Ano novo
if [Data] = #date([Ano],01,01) then "NÃO" else
//Verifica Tiradentes
if [Data] = #date([Ano],04,21) then "NÃO" else
//Verifica dia do Trabalhador
if [Data] = #date([Ano],05,01) then "NÃO" else
//Verifida dia da idenpendencia
if [Data] = #date([Ano],09,07) then "NÃO" else
//Verifica dia de nossa senhora aparecida
if [Data] = #date([Ano],10,12) then "NÃO" else
//verifica Finados
if [Data] = #date([Ano],11,02) then "NÃO" else
//Verifica Proclamação da República
if [Data] = #date([Ano],11,15) then "NÃO" else
//Verifica Natal
if [Data] = #date([Ano],12,25) then "NÃO" else

//Feriados móveis

//Verifica Pascoa 
if [Data] = Date.From(Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6) then "NÃO" else
//Verifica Carnaval
if [Data] = Date.From((Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6 - 47)) then "NÃO" else
//Verifica Sexta - Feira Santa 
if [Data] = Date.From((Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6 - 2)) then "NÃO" else 
//Verifica Corpus Christ
if [Data] = Date.From((Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6 + 60)) then "NÃO" else "SIM", type text),
    #"Nome Feriado Inserido" = Table.AddColumn(#"Personalização Adicionada2", "Nome Feriado", each //Feriados Fixos


//Verifica Ano novo
if [Data] = #date([Ano],01,01) then "Ano Novo" else
//Verifica Tiradentes
if [Data] = #date([Ano],04,21) then "Tiradentes" else
//Verifica dia do Trabalhador
if [Data] = #date([Ano],05,01) then "Dia do Trabalhador" else
//Verifida dia da independencia
if [Data] = #date([Ano],09,07) then "Dia da Independência" else
//Verifica dia de nossa senhora aparecida
if [Data] = #date([Ano],10,12) then "Dia de Nossa Senhora Aparecida" else
//verifica Finados
if [Data] = #date([Ano],11,02) then "Finados" else
//Verifica Proclamação da República
if [Data] = #date([Ano],11,15) then "Proclamação da República" else
//Verifica Natal
if [Data] = #date([Ano],12,25) then "Natal" else

//Feriados móveis
if [Data] = Date.From(Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6) then "Páscoa" else
//Verifica Carnaval
if [Data] = Date.From((Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6 - 47)) then "Carnaval" else
//Verifica Sexta - Feira Santa 
if [Data] = Date.From((Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6 - 2)) then "Sexta-Feira Santa" else 
//Verifica Corpus Christ
if [Data] = Date.From((Number.Round(Number.From(#date([Ano],4,1))/7+Number.Mod(19*Number.Mod([Ano],19)-7,30)*0.14,0)*7-6 + 60)) then "Corpus Christ" else


//Verifica Sabado e Domingo
if [Dia da Semana] = 0 then "Domingo" else
if [Dia da Semana] = 6 then "Sábado" else "Dia Normal", type text)
in
    #"Nome Feriado Inserido"
    

# Terceira forma

let
	// Data Inicial
	DataMin = #date(2011,1,1),
	// Ultima data do mês atual
	Datamax = #date(2019, 12, 31),
	// Quantidade de dias entre DataMin e Datamax
	Qtde_Dias = Duration.TotalDays(Datamax-DataMin)+1,
	// Lista de Datas
	Fonte = List.Dates(DataMin,Qtde_Dias,#duration(1,0,0,0)),
	//Tabela Calendário
	Tabela = #table(type table[Data=date, Dia=Int8.Type,  Semana = text, Semana Abrev = text, Mes Nome= text, Trimestre = text,  Ano = Int16.Type, AnoMes = Int32.Type, MesAno = text],
	List.Transform(Fonte, each {
		_,//data
		Date.Day(_),//dia
		Text.Repeat(Character.FromNumber(8203), 7 - Date.DayOfWeek(_)) & Text.BeforeDelimiter(Text.Proper(Date.ToText(_,"dddd")),"-"),//Nome da Semana
			Text.Repeat(Character.FromNumber(8203), 7 - Date.DayOfWeek(_)) & Text.Proper(Date.ToText(_,"ddd")),//Nome da Semana Abreviado
		Text.Repeat(Character.FromNumber(8203), 12 - Date.Month(_)) & Text.Proper(Date.ToText(_,"MMM")),//Nome do Mes
		"T" & Text.From(Date.QuarterOfYear(_)), //Trimestre
		Date.Year(_),//Ano
		Int32.From(Date.ToText(_,"yyyyMM")), //Ano Mês 202201, 202202, ...
			Text.Proper(Date.ToText(_,"MMM/yyyy")) //Ano Mês Jan/2022, Fev/2022, ...
		}))
in
	Tabela
