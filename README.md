# Batalha-Naval

import java.io.File
import java.io.PrintWriter

var numLinhas = -1
var numColunas = -1

var tabuleiroHumano: Array<Array<Char?>> = emptyArray()
var tabuleiroComputador: Array<Array<Char?>> = emptyArray()
var tabuleiroPalpitesDoHumano: Array<Array<Char?>> = emptyArray()
var tabuleiroPalpitesDoComputador: Array<Array<Char?>> = emptyArray()

const val MENU_PRINCIPAL = 100
const val MENU_DEFINIR_TABULEIRO = 101
const val MENU_DEFINIR_NAVIOS = 102
const val MENU_JOGAR = 103
const val MENU_LER_FICHEIRO = 104
const val MENU_GRAVAR_FICHEIRO = 105
const val SAIR = 106

fun menuLer(): Int{
    println("Introduza o nome do ficheiro (ex: jogo.txt)")
    val ficheiro = readln()
    lerJogo(ficheiro,1)
    lerJogo(ficheiro,2)
    lerJogo(ficheiro,3)
    lerJogo(ficheiro,4)
    println("Tabuleiro ${lerJogo(ficheiro,1).size}x${lerJogo(ficheiro,1).size} lido com sucesso")
    for (mapa in obtemMapa(tabuleiroHumano, true)) println(mapa)
    return MENU_PRINCIPAL
}

fun menuPrincipal(): Int{
    val escolhas: Int
    println("\n> > Batalha Naval < <")
    println("\n1 - Definir Tabuleiro e Navios\n2 - Jogar\n3 - Gravar\n4 - Ler\n0 - Sair\n")
    var escolha = readln().toIntOrNull()
    while (escolha !in (0..4)) {
        println("!!! Opcao invalida, tente novamente")
        escolha = readln().toIntOrNull()
    }
    escolhas = when (escolha){
        0 -> SAIR
        1 -> MENU_DEFINIR_TABULEIRO
        2 -> MENU_JOGAR
        3 -> MENU_GRAVAR_FICHEIRO
        4 -> MENU_LER_FICHEIRO
        else -> MENU_PRINCIPAL
    }
    return escolhas
}

fun menuDefenirNavios(): Int{
    val calculaNavios = calculaNumNavios(numLinhas, numColunas)
    for (dimensao in 1..calculaNavios.size){
        var naviosRestantes = calculaNavios[dimensao - 1]
        while (naviosRestantes > 0){
            val navio = when (dimensao){
                1 -> "submarino:"
                2 -> "contra-torpedeiro:"
                3 -> "navio-tanque:"
                else -> "porta-avioes:"
            }
            println("Insira as coordenadas de um $navio\nCoordenadas (ex: 6,G)")
            var coordenadas = readln()
            if (coordenadas == "-1") return MENU_PRINCIPAL
            while (processaCoordenadas(coordenadas, numLinhas, numColunas) == null){
                println("Coordenadas invalidas, tente novamente\nCoordenadas (ex: 6,G)")
                coordenadas = readln()
            }
            if (dimensao != 1){
                println("Insira a orientacao do navio:\nOrientacao? (N, S, E, O)")
                var orientacao = readln()
                if (orientacao == "-1") return MENU_PRINCIPAL

                while (!(orientacao == "N" || orientacao == "S" || orientacao == "E" || orientacao == "O")) {
                    println("!!! Orientacao invalida, tente novamente\nOrientacao? (N, S, E, O)")
                    orientacao = readln()
                }
                val coordenadaHumano = processaCoordenadas(coordenadas, numLinhas,numColunas)
                if (insereNavio(tabuleiroHumano, coordenadaHumano!!.first , coordenadaHumano.second,orientacao,dimensao)){
                    naviosRestantes--
                    for (posicao in (obtemMapa(tabuleiroHumano, true))) {
                        println(posicao)
                    }
                } else {
                    println("!!! Posicionamento invalido, tente novamente.")
                }
            }else{
                val coordHumano = processaCoordenadas(coordenadas, numLinhas, numColunas)
                if (insereNavioSimples(tabuleiroHumano, coordHumano!!.first, coordHumano.second, dimensao)) {
                    naviosRestantes--
                    for (posicao in (obtemMapa(tabuleiroHumano, true))) {
                        println(posicao)
                    }
                } else {
                    println("!!! Posicionamento invalido, tente novamente.")
                }
            }
        }
    }
    println("Pretende ver o mapa gerado para o Computador? (S/N)")
    val mostrarMapa = readln()
    if (mostrarMapa == "-1") return MENU_PRINCIPAL

    if (mostrarMapa == "S") {
        for (mapa in obtemMapa(tabuleiroComputador, true)) println(mapa)
    }
    return MENU_PRINCIPAL
}

fun calculaNumNavios(numLinhas: Int, numColunas: Int): Array<Int>{
    if (numLinhas == numColunas){
        return when (numLinhas){
            4 ->  arrayOf(2,0,0,0)
            5 ->  arrayOf(1,1,1,0)
            7 ->  arrayOf(2,1,1,1)
            8 ->  arrayOf(2,2,1,1)
            10 ->  arrayOf(3,2,1,1)
            else -> arrayOf()
        }
    }
    return arrayOf()
}

fun criaTabuleiroVazio(numLinhas: Int, numColunas: Int): Array<Array<Char?>>{
    return Array(numLinhas){Array(numColunas){null}}
}

fun coordenadaContida(tabuleiro: Array<Array<Char?>>, linha: Int, coluna: Int): Boolean{
    return linha in 1..tabuleiro.size && coluna in 1..tabuleiro[0].size
}

fun limparCoordenadasVazias(array: Array<Pair<Int,Int>>): Array<Pair<Int,Int>>{
    var resultado: Array<Pair<Int,Int>> = arrayOf()
    for(posicao in array.indices){
        if (array[posicao] != Pair(0,0)){
            resultado += array[posicao]
        }
    }
    return resultado
}

fun juntarCoordenadas(listaCoordenadas: Array<Pair<Int,Int>>, coordenada: Array<Pair<Int,Int>>): Array<Pair<Int,Int>>{
    return listaCoordenadas + coordenada
}

fun gerarCoordenadasNavio(tabuleiro: Array<Array<Char?>>, linha: Int, coluna: Int, orientacao: String, dimensao: Int): Array<Pair<Int,Int>> {
    val coordenadasNavio = Array(dimensao) { Pair(0, 0) }
    val mexerCol = when (orientacao){
        "E" -> 1
        "O" -> -1
        else -> 0
    }
    val mexerLinhas = when (orientacao){
        "N" -> -1
        "S" -> 1
        else -> 0
    }

    for (num in 0 until dimensao) {
        if (coordenadaContida(tabuleiro, linha + num * mexerLinhas, coluna + num * mexerCol)){
            coordenadasNavio[num] = Pair(linha + num * mexerLinhas, coluna + num * mexerCol)
        } else {
            return emptyArray()
        }
    }
    return coordenadasNavio
}

fun eliminarCoordenadasDuplicadas(coordenadas1: Array<Pair<Int, Int>>, coordenadas2: Array<Pair<Int, Int>>): Array<Pair<Int, Int>> {
    for (coordenada in coordenadas1) {
        for (count in coordenadas2.indices) {
            if (coordenadas2[count] == coordenada) {
                coordenadas2[count] = Pair(0, 0)
            }
        }
    }
    return juntarCoordenadas(coordenadas1, limparCoordenadasVazias(coordenadas2))
}

fun gerarCoordenadasFronteira(tabuleiro: Array<Array<Char?>>, linha: Int, coluna: Int, orientacao: String, dimensao: Int): Array<Pair<Int,Int>>{
    var coordenadasFronteira: Array<Pair<Int, Int>> = arrayOf()
    val coordenadasNavio = gerarCoordenadasNavio(tabuleiro, linha, coluna, orientacao, dimensao)
    val movimentacoes = arrayOf(-1, 0, 1)
    for (linhaDeslocamento in movimentacoes) {
        for (colunaDeslocamento in movimentacoes) {
            val novaLinha = linha + linhaDeslocamento
            val novaColuna = coluna + colunaDeslocamento
            if ((Pair(novaLinha, novaColuna) !in coordenadasNavio) && (coordenadaContida(tabuleiro, novaLinha, novaColuna))) {
                coordenadasFronteira += Pair(novaLinha, novaColuna)
            }
        }
    }
    if (dimensao > 1) {
        when (orientacao) {
            "N" -> coordenadasFronteira += (gerarCoordenadasFronteira(tabuleiro, linha - 1, coluna, orientacao, dimensao - 1))
            "S" -> coordenadasFronteira += (gerarCoordenadasFronteira(tabuleiro, linha + 1, coluna, orientacao, dimensao - 1))
            "E" -> coordenadasFronteira += (gerarCoordenadasFronteira(tabuleiro, linha, coluna + 1, orientacao, dimensao - 1))
            "O" -> coordenadasFronteira += (gerarCoordenadasFronteira(tabuleiro, linha, coluna - 1, orientacao, dimensao - 1))
        }
    }
    return limparCoordenadasVazias(eliminarCoordenadasDuplicadas(coordenadasFronteira,coordenadasNavio))
}

fun estaLivre(tabuleiro: Array<Array<Char?>>, coordenadas: Array<Pair<Int, Int>>): Boolean {
    for (pair in coordenadas){
        when{
            (tabuleiro[pair.first -1][pair.second -1] != null || !coordenadaContida(tabuleiro, pair.first, pair.second)) -> return false
        }
    }
    return true
}

fun insereNavioSimples(tabuleiro: Array<Array<Char?>>, linha: Int, coluna: Int, dimensao: Int): Boolean {
    val coordsNavio = gerarCoordenadasNavio(tabuleiro, linha, coluna, "E", dimensao)
    val coordsOcupadas = juntarCoordenadas(gerarCoordenadasNavio(tabuleiro, linha, coluna, "E", dimensao),
        gerarCoordenadasFronteira(tabuleiro, linha, coluna, "E", dimensao))
    if (!estaLivre(tabuleiro, coordsOcupadas)|| coordsNavio.isEmpty()) return false
    for (coords in coordsNavio) tabuleiro[coords.first - 1][coords.second - 1] = dimensao.digitToChar()
    return true
}

fun insereNavio(tabuleiro: Array<Array<Char?>>, linha: Int, coluna: Int, orientacao: String, dimensao: Int): Boolean{
    val coordsNavio = gerarCoordenadasNavio(tabuleiro, linha, coluna, orientacao, dimensao)
    val coordsOcupadas = juntarCoordenadas(gerarCoordenadasNavio(tabuleiro, linha, coluna, orientacao, dimensao),
        gerarCoordenadasFronteira(tabuleiro, linha, coluna, orientacao, dimensao))
    if (!estaLivre(tabuleiro, coordsOcupadas)|| coordsNavio.isEmpty()) return false
    for (coords in coordsNavio) tabuleiro[coords.first - 1][coords.second - 1] = dimensao.digitToChar()
    return true
}

fun preencheTabuleiroComputador(tabuleiro: Array<Array<Char?>>, navios: Array<Int>){
    for (tamanhoNavio in navios.indices) {
        var naviosRestantes = navios[tamanhoNavio]
        val direcoes = arrayOf("N", "S", "E", "O")
        while (naviosRestantes>0) {
            val linha = (0 until tabuleiro.size).random()
            val coluna = (0 until tabuleiro[0].size).random()
            val direcao = direcoes.random()
            if (insereNavio(tabuleiro, linha + 1, coluna + 1, direcao, tamanhoNavio + 1)){
                naviosRestantes--
            }
        }
    }
}

fun verificarNavioNaDirecao (tabuleiro: Array<Array<Char?>>, linha: Int, coluna: Int, direcaoLinha: Int, direcaoColuna: Int):Boolean{
    val navioAtual = tabuleiro[linha - 1][coluna - 1]
    var linhaAtual = linha
    var colunaAtual = coluna
    var tamanhoNavio = '0'

    while (coordenadaContida(tabuleiro, linhaAtual, colunaAtual) &&
        tabuleiro[linhaAtual - 1][colunaAtual - 1] == navioAtual){
        tamanhoNavio++
        linhaAtual += direcaoLinha
        colunaAtual += direcaoColuna
    }

    linhaAtual = linha - direcaoLinha
    colunaAtual = coluna - direcaoColuna

    while (coordenadaContida(tabuleiro, linhaAtual, colunaAtual) &&
        tabuleiro[linhaAtual - 1][colunaAtual - 1] == navioAtual){
        tamanhoNavio++
        linhaAtual -= direcaoLinha
        colunaAtual -= direcaoColuna
    }
    return tamanhoNavio == (navioAtual)
}

fun navioCompleto(tabuleiro: Array<Array<Char?>>, linha: Int, coluna: Int): Boolean{
    return (coordenadaContida(tabuleiro, linha, coluna) &&
            verificarNavioNaDirecao(tabuleiro, linha,coluna,0,1) ||
            verificarNavioNaDirecao(tabuleiro, linha,coluna,1,0))
}

fun obtemMapa(tabuleiro: Array<Array<Char?>>, tabuleiroReal: Boolean): Array<String>{
    val linhas = tabuleiro.size
    val colunas = tabuleiro[0].size
    val mapa = Array(linhas + 1){ "" }
    mapa[0]="| ${criaLegendaHorizontal(colunas)} |"
    for (linha in 1..linhas) {
        for (coluna in 1..colunas) {
            val navio = tabuleiro[linha - 1][coluna - 1]
            if (navio == null) {
                mapa[linha] += if (tabuleiroReal) {
                    "| ~ "
                } else {
                    "| ? "
                }
            } else if (!tabuleiroReal) {
                val num = when {
                    navio == '2' && !navioCompleto(tabuleiro, linha, coluna) -> '\u2082'
                    navio == '3' && !navioCompleto(tabuleiro, linha, coluna) -> '\u2083'
                    navio == '4' && !navioCompleto(tabuleiro, linha, coluna) -> '\u2084'
                    else -> navio
                }
                mapa[linha] += "| $num "
            } else {
                mapa[linha] += "| $navio "
            }
        }
        mapa[linha] += "| $linha"
    }
    return mapa
}

fun lancarTiro(tabuleiroReal: Array<Array<Char?>>, tabuleiroPalpites: Array<Array<Char?>>, coordenadas: Pair<Int,Int>): String{
    if (tabuleiroReal[coordenadas.first - 1][coordenadas.second - 1] != null){
        tabuleiroPalpites[coordenadas.first - 1][coordenadas.second - 1] = tabuleiroReal[coordenadas.first - 1][coordenadas.second - 1]
        val navio = when (tabuleiroReal[coordenadas.first - 1][coordenadas.second - 1]){
            '1' -> "submarino"
            '2' -> "contra-torpedeiro"
            '3' -> "navio-tanque"
            else -> "porta-avioes"
        }
        return "Tiro num $navio."
    }
    tabuleiroPalpites[coordenadas.first - 1][coordenadas.second - 1] = 'X'
    return "Agua."
}

fun geraTiroComputador(tabuleiroPalpitesComputador: Array<Array<Char?>>): Pair<Int, Int>{
    val coords = Array(tabuleiroPalpitesComputador.size * tabuleiroPalpitesComputador.size){Pair(0,0)}
    var casa = 0
    for (linha in tabuleiroPalpitesComputador.indices){
        for (coluna in tabuleiroPalpitesComputador[0].indices){
            if (tabuleiroPalpitesComputador[linha][coluna] == null){
                coords[casa] = Pair(linha + 1, coluna + 1)
                casa++
            }
        }
    }
    val coordenadasNull = limparCoordenadasVazias(coords)
    return coordenadasNull.random()
}

fun marcarNavioContado(linha: Int, coluna: Int, dimensao: Int) : Array<Pair<Int, Int>> {
    var naviosPassados: Array<Pair<Int, Int>> = arrayOf()
    for (num in 0 until dimensao) {
        naviosPassados += (Pair(linha + num, coluna))
    }
    for (num in 0 until dimensao) {
        naviosPassados += (Pair(linha, coluna + num))
    }
    for (num in 0 until dimensao) {
        naviosPassados += (Pair(linha - num, coluna))
    }
    for (num in 0 until dimensao) {
        naviosPassados += (Pair(linha, coluna - num))
    }
    return naviosPassados
}

fun contarNaviosDeDimensao(tabuleiro: Array<Array<Char?>>, dimensao: Int): Int {
    var contadorNavios = 0
    var naviosContados: Array<Pair<Int, Int>> = arrayOf()

    val tamanho: Char = when (dimensao) {
        1 -> '1'
        2 -> '2'
        3 -> '3'
        else -> '4'
    }

    for (linha in 1..tabuleiro.size) {
        for (coluna in 1..tabuleiro[linha - 1].size) {
            if (tabuleiro[linha - 1][coluna - 1] == tamanho) {
                val posicao = Pair(linha, coluna)
                if (posicao !in naviosContados &&
                    navioCompleto(tabuleiro, linha, coluna)
                ) {
                    naviosContados += marcarNavioContado(linha, coluna, dimensao)
                    contadorNavios++
                }
            }
        }
    }
    return contadorNavios
}

fun venceu(tabuleiro: Array<Array<Char?>>): Boolean{
    val numNavios = calculaNumNavios(tabuleiro.size, tabuleiro[0].size)

    for (dimensao in 1..numNavios.size){
        val naviosEsperados = numNavios[dimensao - 1]
        val naviosAchados = contarNaviosDeDimensao(tabuleiro, dimensao)

        if (naviosEsperados != naviosAchados) return false
    }
    return true
}

fun lerJogo(ficheiro: String, tipoTabuleiro: Int): Array<Array<Char?>> {
    val ficheiroGravado = File(ficheiro).readLines()

    numLinhas = ficheiroGravado[0][0].digitToInt()
    numColunas = ficheiroGravado[0][2].digitToInt()

    val linhasNoFicheiro = 3 * tipoTabuleiro + 1 + numLinhas * (tipoTabuleiro - 1) until 3 * tipoTabuleiro + 1 + numLinhas * tipoTabuleiro
    val tabuleiro = criaTabuleiroVazio(numLinhas,numColunas)
    var linha = linhasNoFicheiro.min()
    while(linha in linhasNoFicheiro){
        var coluna = 0
        var contaVirgulas = 0
        while (contaVirgulas < numColunas && coluna < ficheiroGravado[linha].length){
            when (ficheiroGravado[linha][coluna]){
                ',' -> contaVirgulas++
                '1','2','3','4','X' -> tabuleiro[linha - linhasNoFicheiro.min()][contaVirgulas] = ficheiroGravado[linha][coluna]
            }
            coluna++
        }
        linha++
    }
    when(tipoTabuleiro) {
        1 -> tabuleiroHumano = tabuleiro
        2 -> tabuleiroPalpitesDoHumano = tabuleiro
        3 -> tabuleiroComputador = tabuleiro
        4 -> tabuleiroPalpitesDoComputador = tabuleiro
    }
    return tabuleiro
}

fun imprimirTabuleiro(filePrinter: PrintWriter, tabuleiro: Array<Array<Char?>>) {
    for (linha in tabuleiro) {
        var count = 1
        for (coluna in linha) {
            if (count != tabuleiro.size) {
                filePrinter.print("${coluna ?: ""},")
                count++
            } else {
                filePrinter.print("${coluna ?: ""}")
            }
        }
        filePrinter.println()
    }
}

fun gravarJogo(ficheiro: String, tabuleiroRealHumano: Array<Array<Char?>>, tabuleiroPalpitesDoHumano: Array<Array<Char?>>,
               tabuleiroRealComputador: Array<Array<Char?>>, tabuleiroPalpitesDoComputador: Array<Array<Char?>>) {
    val filePrinter = File(ficheiro).printWriter()

    filePrinter.println("${tabuleiroRealHumano.size},${tabuleiroRealHumano[0].size}\n")

    filePrinter.println("Jogador\nReal")
    imprimirTabuleiro(filePrinter, tabuleiroRealHumano)

    filePrinter.println("\nJogador\nPalpites")
    imprimirTabuleiro(filePrinter, tabuleiroPalpitesDoHumano)

    filePrinter.println("\nComputador\nReal")
    imprimirTabuleiro(filePrinter, tabuleiroRealComputador)

    filePrinter.println("\nComputador\nPalpites")
    imprimirTabuleiro(filePrinter, tabuleiroPalpitesDoComputador)

    filePrinter.close()
}

fun opcaoJogar(): Int {
    if(tabuleiroHumano.isEmpty() && tabuleiroComputador.isEmpty()) {
        println("!!! Tem que primeiro definir o tabuleiro do jogo, tente novamente")
    } else {
        while (true) {
            for (mapa in obtemMapa(tabuleiroPalpitesDoHumano, false)) println(mapa)
            var coordenadas: Pair<Int, Int>?
            do {
                println("Indique a posição que pretende atingir\nCoordenadas? (ex: 6,G)")
                val leitura = readln()
                if (leitura == "-1"){
                    return MENU_PRINCIPAL
                }
                coordenadas = processaCoordenadas(leitura, numLinhas, numColunas)
                if (coordenadas == null || !estaLivre(tabuleiroPalpitesDoHumano, arrayOf(coordenadas))) {
                    println(
                        "!!! Insira coordenadas que estejam contidas no tabuleiro e numa posição ainda não atingida, no formato correto.\n"
                    )
                }
            } while (coordenadas == null || !estaLivre(tabuleiroPalpitesDoHumano, arrayOf(coordenadas)))
            print(">>> HUMANO >>>${lancarTiro(tabuleiroComputador, tabuleiroPalpitesDoHumano, coordenadas)}")
            if (navioCompleto(tabuleiroPalpitesDoHumano, coordenadas.first, coordenadas.second)) {
                print(" Navio ao fundo!")
            }
            var venceu = false
            if (venceu(tabuleiroPalpitesDoHumano)) {
                println("\nPARABENS! Venceu o jogo!")
                venceu = true
            } else {
                println()
                val tiroComputador = geraTiroComputador(tabuleiroPalpitesDoComputador)
                println("Computador lançou tiro para a posição $tiroComputador")
                print(">>> COMPUTADOR >>>${lancarTiro(tabuleiroHumano, tabuleiroPalpitesDoComputador, tiroComputador)}")
                if (navioCompleto(tabuleiroPalpitesDoHumano, coordenadas.first, coordenadas.second)) {
                    print(" Navio ao fundo!")
                }
                println()
            }

            if (venceu(tabuleiroPalpitesDoComputador)) {
                println("OPS! O computador venceu o jogo!")
                venceu = true
            }
            if (venceu) {
                tabuleiroHumano = emptyArray()
                tabuleiroComputador = emptyArray()
                tabuleiroPalpitesDoHumano = emptyArray()
                tabuleiroPalpitesDoComputador = emptyArray()
                println("Prima enter para voltar ao menu principal")
                return MENU_PRINCIPAL
            }
            var escolha: String?
            println("Prima enter para continuar")
            escolha = readLine()
            if (escolha == "-1") {
                return MENU_PRINCIPAL
            }
        }
    }
    return MENU_PRINCIPAL
}

fun processaCoordenadas(coordenadas:String?, numLinhas: Int, numColunas: Int): Pair<Int, Int>? {
    if (coordenadas == null) return null
    if (coordenadas == "") return null
    if ((coordenadas.length == 4) && (coordenadas[2] != ',')) return null

    val valorY = coordenadas[coordenadas.length-1].toInt()-64

    val valorX: Int = if (coordenadas.length == 4){
        "${coordenadas[0]}${coordenadas[1]}".toInt()
    }else {
        coordenadas[0].toInt()-48
    }
    if (valorX > numLinhas || valorX < 1){
        return null
    }
    if (valorY > numColunas || valorY < 1) {
        return null
    }
    return Pair(valorX, valorY)
}

fun criaLegendaHorizontal(numColunas: Int): String {
    var numAtual = 0
    var legenda = ""
    do {
        numAtual++
        legenda += (numAtual+64).toChar()
        if (numAtual < numColunas){
            legenda += (" | ")
        }
    } while (numAtual < numColunas)
    return legenda
}

fun criaTerreno(numLinhas: Int, numColunas: Int): String{
    val linhas = "| ~ "
    var numeros = 0
    var legenda = ""
    var num2 = 0
    var legendafin = "\n| " + criaLegendaHorizontal(numColunas) + " |\n"
    do {
        numeros++
        legenda += linhas
    } while (numeros != numColunas)
    do {
        num2++
        legendafin += "$legenda| $num2\n"
    } while (num2 <= numLinhas - 1)
    return legendafin
}

fun menuDeJogo():String{
    return "\n> > Batalha Naval < <\n" +
            "\n1 - Definir Tabuleiro e Navios" +
            "\n2 - Jogar" +
            "\n3 - Gravar" +
            "\n4 - Ler" +
            "\n0 - Sair\n"
}

fun tamanhoTabuleiroValido(numLinhas: Int?, numColunas: Int?): Boolean {
    return when{
        numLinhas != numColunas -> false
        numLinhas == null -> false
        numLinhas == 4 -> true
        numLinhas == 5 -> true
        numLinhas == 7 -> true
        numLinhas == 8 -> true
        numLinhas == 10 -> true
        else -> false
    }
}

fun defenirTabuleiro(): Int{
    var linhas: Int?
    var colunas: Int?
    println("\n> > Batalha Naval < <\n" +
            "\nDefina o tamanho do tabuleiro:")
    do {
        do {
            println("Quantas linhas?")
            linhas = readln().toIntOrNull()
            when (linhas) {
                null -> println("Numero de linhas invalidas, tente novamente")
                -1 -> return MENU_PRINCIPAL
            }
        } while (linhas == null)
        do {
            println("Quantas colunas?")
            colunas = readln().toIntOrNull()
            when (colunas){
                null -> println("Numero de colunas invalidas, tente novamente")
                -1 -> return MENU_PRINCIPAL
            }
        } while (colunas == null)
    } while (!tamanhoTabuleiroValido(linhas, colunas))
    if (linhas == null || colunas == null){
        return MENU_PRINCIPAL
    }
    numLinhas = linhas
    numColunas = colunas
    tabuleiroHumano = criaTabuleiroVazio(numLinhas, numColunas)
    tabuleiroComputador = criaTabuleiroVazio(numLinhas, numColunas)
    tabuleiroPalpitesDoHumano = criaTabuleiroVazio(numLinhas, numColunas)
    tabuleiroPalpitesDoComputador = criaTabuleiroVazio(numLinhas, numColunas)

    for (mapa in obtemMapa(tabuleiroHumano, true)) println(mapa)
    return MENU_DEFINIR_NAVIOS
}

fun inserirBarco(numLinhas: Int, numColunas: Int) {
    println("\nInsira as coordenadas do navio:")

    do {
        println("Coordenadas? (ex: 6,G)")
        val coordenadas = readLine()
        if (coordenadas == "-1") return
        if (processaCoordenadas(coordenadas, numLinhas, numColunas) == null) println("!!! Coordenadas invalidas, tente novamente")
    } while (processaCoordenadas(coordenadas, numLinhas, numColunas) == null)

    var orientacaoInvalida: Boolean

    println("Insira a orientacao do navio:")
    do {
        println("Orientacao? (N, S, E, O)")
        val orientacao = readLine()
        if (orientacao == "-1") return
        orientacaoInvalida = (orientacao != "N" && orientacao != "S" && orientacao != "E" && orientacao != "O")
        if (orientacaoInvalida) println("!!! Orientacao invalida, tente novamente")
    } while (orientacaoInvalida)
}

fun escolhaOpcao(): Int{
    var opcao:Int?
    do {
        opcao = readLine()?.toIntOrNull()
        if (opcao == null || opcao !in 0..4) {
            println("!!! Opcao invalida, tente novamente")
            opcao = null
        }
    }while (opcao == null)
    return opcao
}

fun menuGravarFicheiro(): Int{
    if(tabuleiroHumano.isEmpty() && tabuleiroComputador.isEmpty()){
        println("!!! Tem que definir o tabuleiro do jogo, tente novamente")
        return MENU_PRINCIPAL
    }
    println("Introduza o nome do ficheiro (ex: jogo.txt)")
    val ficheiro = readln()
    gravarJogo(ficheiro,tabuleiroHumano,tabuleiroPalpitesDoHumano,tabuleiroComputador,tabuleiroPalpitesDoComputador)
    println("Tabuleiro ${tabuleiroHumano.size}x${tabuleiroHumano[0].size} gravado com sucesso")
    return MENU_PRINCIPAL
}

fun main() {
    var menuAtual = MENU_PRINCIPAL
    while (true) {
        menuAtual = when (menuAtual) {
            MENU_PRINCIPAL -> menuPrincipal()
            MENU_DEFINIR_TABULEIRO -> defenirTabuleiro()
            MENU_DEFINIR_NAVIOS -> menuDefenirNavios()
            MENU_JOGAR -> opcaoJogar()
            MENU_GRAVAR_FICHEIRO -> menuGravarFicheiro()
            MENU_LER_FICHEIRO -> menuLer()
            SAIR -> return
            else -> return
        }
    }
}
