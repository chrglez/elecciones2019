Este 28 de abril de 2019 se celebraron en España elecciones generales
para elegir los representantes al congreso y senado de la XII
legislatura. Los miembros del congreso se eligen mediante un sistema de
representación proporcional. El número actual de representantes es de
350 diputados \[[Ley Orgánica del Régimen Electoral
General](https://es.wikipedia.org/wiki/Legislaci%C3%B3n_electoral_espa%C3%B1ola)\]

Existen 52 circunscripciones electorales para el Congreso de los
Diputados, que se corresponden con las cincuenta provincias españolas
más las ciudades autónomas de Ceuta y Melilla. Según la ley electoral
española cada provincia tiene garantizado un mínimo de partida de dos
escaños y las ciudades autónomas de Ceuta y Melilla, uno cada una de
ellas. Los otros 248 diputados se asignan de forma proporcional a la
población de derecho. Los escaños a las listas electorales en cada
circunscripción se reparten usanso el sistema D’Hondt en cada
circunscripción.

Los resultados provisionales pueden consultarse en la
[web](https://resultados.eleccionesgenerales19.es/Congreso/Total-nacional/0/es)
que el Ministerio del Interior ha creado al efecto.

Quise aprovechar para utilizar el paquete
[\#ggparliament](https://github.com/RobWHickman/ggparliament) y replicar
los resultados.

    ## Warning: package 'ggplot2' was built under R version 3.5.1

    ## Warning: package 'dplyr' was built under R version 3.5.1

    ## Warning: package 'ggparliament' was built under R version 3.5.3

![](elecciones_md_files/figure-markdown_github/unnamed-chunk-1-1.png)

    ## Warning: Column `Provincia` joining factor and character vector, coercing
    ## into character vector

Repartidos por circunscripciones quedaría distribuida de la siguiente
manera

``` r
ggplot(mapa_elec, aes(x = x, y = y)) +
  geom_point(size = 3) +
  geom_point(aes(color = seats_short), size = 2) +
  scale_color_manual(values = 
                       c('#E3060F','#0BB2F7','#EB6109','#6A1F5F',
                               '#68BD00','#FEB832','#ED5975','#417505',
                               '#B9FF71','#FFE605','#E62C25','#D54531',
                               '#C1CD01', '#000000')) +
  theme_void() +
  theme(legend.position = 'bottom',
        legend.title = element_blank())
```

<img src="elecciones_md_files/figure-markdown_github/unnamed-chunk-3-1.png" style="display: block; margin: auto;" />

En forma de gráfico de barras

![](elecciones_md_files/figure-markdown_github/unnamed-chunk-4-1.png)

Cambiando las reglas del juego
==============================

Siempre he querido experimentar como hubiesen sido los resultados con
otros sistema de reparto como el de circunscripción único (lo que
equivaldría a que el peso de un voto sería el mismo para todas las
provincias) o incluso si utilizasen los sistemas empleados en otros
países para la elección de representantes, como el de Reino Unido, o de
presidente, como el de Estados Unidos.

Cincunscripción única
---------------------

En este caso suponemos que existe una única cincuscripción para toda
Espña. De esta forma, se aplicaría el sistema de reparto d’Hont para los
votos de los diferentes partidos obtenidos en toda España. El parlamento
quedaría formado de la siguiente manera.

![](elecciones_md_files/figure-markdown_github/unnamed-chunk-5-1.png) En
forma de gráfico de barras

![](elecciones_md_files/figure-markdown_github/unnamed-chunk-6-1.png)
\#\# Sistema de elecciones presidenciales en los Estados Unidos.

Hay que recordar que en las elecciones generales en España los electores
no eligen presidente sino representantes para el Congreso y el Senado.
Es en la investidura donde el Parlamento reconoce y acepta al nuevo
presidente del Gobierno. En Estados Unidos el presidente se elige de
forma indirecta. Cada Estado tiene asignado un número de compromisarios,
que son los que realmente se eligen en las elecciones presidenciales y
que mediante sus votos (votos electorales) eligen presidente. La mayoría
de Estados conceden todos sus votos electorales al candidato que gana la
mayoría absuluta de los votos emitidos por los ciudadanos. Es decir, es
un sistema de todo o nada. El que partido qeu gana se queda con todos
los compromisarios de ese Estado.

Extrapolando este sistema al caso español, podríamos equiparar cada de
las 50 cicunscripciones como un Estado de lso Esatdos Unidos y el número
de compromisarios con el número de escaños asigandos a dicha
circunscripción. De esta manera, el partido que gana esa circunscripción
se lleva todos los escaños (compromisarios) asociados a esa
circunscripción. El mapa quedaría configurado de la siguiente manera.

``` r
asignar_asientos_usa <- function(nccaa, nprov){
  
  url <- paste0('https://resultados.elpais.com/elecciones/2019/generales/congreso/',nccaa,'/',nprov,'.html')
  
  
  
  datos <- url %>%
    read_html(url) %>%
    html_nodes("#tablaVotosPartidos") %>%
    html_table() %>%
    `[[`(1) %>%
    filter(`Escaños` >= 1) %>%
    uncount(`Escaños`)
  
  
  seats <- tibble(seats = rep(datos$Partido[1],sum(nrow(datos))),
                  nccaapais = nccaa,
                  cprov = nprov)
  

}

scrape_seats <- purrr::map2_df(cod_provincia$nccaapais,cod_provincia$cprov, asignar_asientos_usa)

seats_scrape <- scrape_seats %>%
  mutate(seats_short = seats %>%
  str_replace_all('^PODEMOS[\\sÚA-Z-]*','UP') %>%
  str_replace_all('^ECP[\\sA-Z-]*','UP') %>%
  str_replace_all('^PSC[\\sA-Z-]*','PSOE') %>%
  str_replace_all('^PSdeG[\\sA-Z-]*','PSOE') %>%
  str_replace_all('^PSE[\\sA-Z-]*','PSOE') %>%
  str_replace_all('^ERC[\\sA-Z-]*','ERC') %>%
    str_replace_all('^Jx[\\sA-Z-]*','JxCAT') %>%
  str_replace_all('PNV','EAJ-PNV')) %>%
  arrange(cprov)

mapa_elec <- rbind(co, lu, po, or, as, s, bi, ss, vi, na,
                   hu, za, te, le, ba, gi, ta, cs, va, ac,
                   pm, mu, al, gr, ma, ca, hl, se, cd,
                   ja, bj, cc, sa, zm, ln, pl, bu, so, vl,
                   sg, av, cr, to, ab, cu, gu, md, lo, tf,
                   lp, ce, ml)

mapa_elec <- mapa_elec %>%
  mutate(Provincia = prov) %>%
  inner_join(cod_provincia) %>%
  mutate(cprov1 = cprov,
         colorprov = as.numeric(cprov) %% 2) %>%
  select(x,y,prov,cprov1, colorprov) %>%
  arrange(cprov1) %>%
  cbind(seats_scrape) %>%
  mutate(seats_short = factor(seats_short,
                              levels = party_col$party_short[party_col$party_short %in% unique(seats_short)]))
```

    ## Joining, by = "Provincia"

    ## Warning: Column `Provincia` joining factor and character vector, coercing
    ## into character vector

``` r
ggplot(mapa_elec, aes(x = x, y = y)) +
  geom_point(size = 3) +
  geom_point(aes(color = seats_short), size = 2) +
  scale_color_manual(values = party_col$color[party_col$party_short %in% unique(mapa_elec$seats_short)]) +
  theme_void() +
  theme(legend.position = 'bottom',
        legend.title = element_blank())
```

<img src="elecciones_md_files/figure-markdown_github/unnamed-chunk-7-1.png" style="display: block; margin: auto;" />

El PSOE tendría más de 176 compromisarios, por lo que Sánchez sería
elegido Presidente del Gobierno sin necesidad de pactos.

![](elecciones_md_files/figure-markdown_github/unnamed-chunk-8-1.png)

Sistema de elección al parlamento en el Reino Unido
---------------------------------------------------

En este caso, las circunscripciones son mucho más pequeñas y el
representante se elige por escrutino uninominal mayoritario. Es decir,
quien gana en ese distrito es designado miembro del parlamento. Para
poder hacer la analogía, he supuesto que cada uno de los escaños para
cada provincia se corresponda con los municipios más poblados de esa
circunscripción. Por ejemplo, para A Coruña con 8 escaños sus
representantes saldrán de los partidos más votados en los 8 municipios
más poblados.

El mapa electoral de España quedaría dibujado de la siguiente manera:

``` r
ci <- mapa_elec %>%
  group_by(prov) %>%
  summarise(n = n())

url <- 'http://www.ine.es/dynt3/inebase/index.htm?padre=525'

page <- url %>%
  read_html()

provincias <- page %>%
  html_nodes('.inebase_tabla .titulo') %>%
  html_text() %>%
  str_remove(':[\\w\\s\\.]*')

enlaces <- page %>%
  html_nodes('.inebase_tabla .titulo') %>%
  html_attr('href') %>%
  str_c('http://www.ine.es',.) %>%
  str_remove('&L=0') %>%
  str_replace('Tabla','Datos')

prov_pob <- tibble(provincias, enlaces_ine = enlaces)
prov_pob[prov_pob$provincias == 'Avila','provincias'] = 'Ávila'

cod_prov_ine <- inner_join(cod_provincia, prov_pob, c('Provincia' = 'provincias'))

ci_melt <- inner_join(ci, prov_pob, c('prov' = 'provincias'))
```

    ## Warning: Column `prov`/`provincias` joining factor and character vector,
    ## coercing into character vector

``` r
populate_muni <- function(prov, n, enlaces_ine){
  
url <- enlaces_ine

datos <- url %>%
    read_html(url) %>%
    html_nodes("#tablaDatos") %>%
    html_table(fill = T) %>%
  `[[`(1) %>%
  tail(-3) %>%
  select(1:2) %>%
  setNames(c('municipio','poblacion')) %>%
  mutate(poblacion = str_remove_all(.$poblacion,'\\.') %>% as.numeric) %>%
  arrange(desc(poblacion)) %>%
  slice(1:n) %>%
  mutate(municipio = str_remove(.$municipio,'[\\s\\d]*'))
output <- tibble(provincia = prov, municipio = datos$municipio)

}

municipios.df <- purrr::pmap_df(ci_melt, populate_muni)

municipios_pais <- inner_join(municipios.df, cod_provincia, c('provincia' = 'Provincia'))

scrape_links_muni_pais <- function(nccaa, nprov){
   url <- paste0('https://resultados.elpais.com/elecciones/2019/generales/congreso/',nccaa,'/',nprov,'.html')
   
   page <- url %>%
     read_html()
   
   municipios <- page %>%
     html_nodes('#listadoMunicipios li') %>%
     html_text()
   
   muni_link <- page %>%
     html_nodes('#listadoMunicipios li a') %>%
     html_attr('href')
   
   output <- tibble(nccaa, nprov, municipios, muni_link)
}

links_muni <- purrr::map2_df(cod_provincia$nccaapais,cod_provincia$cprov, scrape_links_muni_pais)

muni <- municipios_pais$municipio
muni[str_detect(muni,',')] <- str_subset(muni,',') %>% map_chr(~str_split(.x,',') %>% `[[`(1) %>% `[`(c(2,1)) %>% str_c(collapse = ' ') %>% str_trim())
muni[str_detect(muni,'^la ')] <- str_subset(muni,'^la ') %>% map_chr(~str_remove(.x,'^la ') %>% str_c(., ' (la)'))
muni[str_detect(muni,"L' ")] <- str_subset(muni,"L' ") %>% map_chr(~str_replace(.x,"L' ","L'"))
muni <- str_replace_all(muni,'/',' / ')
muni[57] <- 'Barañain'
municipios_pais$municipio <- muni




cod_muni <- left_join(municipios_pais, links_muni, c('municipio' = 'municipios', 'cprov' = 'nprov'))


asignar_asientos <- function(nccaa, muni_link){
  
  url <- paste0('https://resultados.elpais.com/elecciones/2019/generales/congreso/',nccaa,'/',muni_link)
  
  
  
  datos <- url %>%
    read_html(url) %>%
    html_nodes("#tablaVotosPartidos") %>%
    html_table() %>%
    `[[`(1)
  
  
  seats <- tibble(seats = datos$Partido[1],
                  nccaapais = nccaa,
                  cprov = substr(muni_link, start = 1, stop = 2))
  

}

scrape_seats <- purrr::map2_df(cod_muni$nccaapais,cod_muni$muni_link, asignar_asientos)

seats_scrape <- scrape_seats %>%
  mutate(seats_short = seats %>%
  str_replace_all('^PODEMOS[\\sÚA-Z-]*','UP') %>%
  str_replace_all('^ECP[\\sA-Z-]*','UP') %>%
  str_replace_all('^PSC[\\sA-Z-]*','PSOE') %>%
  str_replace_all('^PSdeG[\\sA-Z-]*','PSOE') %>%
  str_replace_all('^PSE[\\sA-Z-]*','PSOE') %>%
  str_replace_all('^ERC[\\sA-Z-]*','ERC') %>%
    str_replace_all('^Jx[\\sA-Z-]*','JxCAT') %>%
  str_replace_all('PNV','EAJ-PNV')) %>%
  arrange(cprov)

mapa_elec <- rbind(co, lu, po, or, as, s, bi, ss, vi, na,
                   hu, za, te, le, ba, gi, ta, cs, va, ac,
                   pm, mu, al, gr, ma, ca, hl, se, cd,
                   ja, bj, cc, sa, zm, ln, pl, bu, so, vl,
                   sg, av, cr, to, ab, cu, gu, md, lo, tf,
                   lp, ce, ml)

mapa_elec <- mapa_elec %>%
  mutate(Provincia = prov) %>%
  inner_join(cod_provincia) %>%
  mutate(cprov1 = cprov,
         colorprov = as.numeric(cprov) %% 2) %>%
  select(x,y,prov,cprov1, colorprov) %>%
  arrange(cprov1) %>%
  cbind(seats_scrape) %>%
  mutate(seats_short = factor(seats_short,
                              levels = party_col$party_short[party_col$party_short %in% unique(seats_short)]))
```

    ## Joining, by = "Provincia"

    ## Warning: Column `Provincia` joining factor and character vector, coercing
    ## into character vector

``` r
ggplot(mapa_elec, aes(x = x, y = y)) +
  geom_point(size = 3) +
  geom_point(aes(color = seats_short), size = 2) +
  scale_color_manual(values = party_col$color[party_col$party_short %in% unique(mapa_elec$seats_short)]) +
  theme_void() +
  theme(legend.position = 'bottom',
        legend.title = element_blank())
```

<img src="elecciones_md_files/figure-markdown_github/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

El parlamento quedaría configurado de la siguiente manera:

    ## Warning: Column `seats_short`/`party_short` joining factor and character
    ## vector, coercing into character vector

![](elecciones_md_files/figure-markdown_github/unnamed-chunk-10-1.png)
En forma de gráfico de barras

    ## Warning: Column `seats_short`/`party_short` joining factor and character
    ## vector, coercing into character vector

![](elecciones_md_files/figure-markdown_github/unnamed-chunk-11-1.png)
