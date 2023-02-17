# Mentoria Digytal Space - Desafio 2

### Requisitos
Tendo como base um banco de dados com uma tabela de produtos e outra de categorias, realizar consultas que respondam:
* qual a quantidade de produtos de cada categoria;
* qual categoria dá mais lucro, considerando a soma do preço de seus produtos;

### Conceitos praticados
* funções de agregação (COUNT, MAX, MIN, SUM, AVG)
* funções de ordenação (ORDER BY, GROUP BY)

### Preparação do ambiente
Para realizar essa atividade, usei os mesmos scripts de criação de banco de dados, tabelas e relacionamentos entre tabelas deste <a href="https://github.com/alonso-estevam/desafio_1">exercício</a>. Modifiquei apenas os inserts, substituindo por produtos e categorias relacionadas ao tema de loja de bicicletas.
Para inserir um número um pouco maior de dados, digitei numa planilha do excel cerca de 50 produtos e a partir disso gerei um arquivo .csv, que em seguida importei no pgAdmin para popular a tabela de produtos.
O modo como fiz a importação do arquivo csv no pgAdmin pode ser visto no vídeo abaixo:

https://user-images.githubusercontent.com/86576674/219758862-84972785-59e1-4cc1-bbdf-061e7d738dd5.mp4

### Aquecimento
Com o ambiente do banco de dados preparado e populado, vamos fazer algumas consultas de agregação, por exemplo, para saber:
* qual o preço do produto **mais caro** da loja, usando a função `MAX`:
```
SELECT MAX(valor_unitario) FROM tb_produtos;
```
* qual o preço do produto **mais barato** da loja, usando a função `MIN`:
```
SELECT MIN(valor_unitario) FROM tb_produtos;
```
* qual a média de preços dos produtos de acordo com a categoria:
```
SELECT categoria, ROUND(AVG(valor_unitario),2) FROM tb_produtos GROUP BY categoria;
```

### Resolução
Depois de exercitar a sintaxe das funções de agregação com exemplos simples, vamos para a resolução das perguntas propostas:
1. Qual a quantidade de produtos de cada categoria?

A função que **conta** as coisas é o `COUNT`, e a função que **agrupa** é o `GROUP BY`. Assim, o comando sql fica:
```
SELECT categoria, COUNT(nome) FROM tb_produtos GROUP BY categoria;
```
2. Qual categoria dá mais lucro, considerando a soma do preço de seus produtos?

Para responder a essa pergunta, precisamos **somar** os preços dos produtos de cada categoria - função `SUM` - e selecionar o **valor máximo** dentre os resultados - função `MAX`. Mas o comando não é tão simples como parece à primeira vista, pois envolve um série de subconsultas. Depois de muito pensar sem conseguir uma resposta, recorri a um grupo dos colegas da faculdade, e o colega <a href="https://github.com/geremiasslima">Geremias Lima</a> veio com essa solução:
```
SELECT tb_categorias.nome, SUM (tb_produtos.valor_unitario) AS soma_dos_precos
FROM tb_produtos
INNER JOIN tb_categorias ON tb_produtos.categoria = tb_categorias.id
GROUP BY tb_categorias.nome
HAVING SUM(tb_produtos.valor_unitario) = (
    SELECT MAX(soma_dos_precos) 
    FROM (
    	SELECT SUM(valor_unitario) as soma_dos_precos
	FROM tb_produtos GROUP BY categoria
	)
	as somas
);
```
De acordo com o colega, a cláusula `INNER JOIN` foi usada para combinar as duas tabelas (produtos e categorias) com base no campo categoria. A função `SUM`, como mencionado, soma o valor de todos os produtos em cada categoria, agrupando os resultados pela coluna `tb_categorias.nome`, da tabela `tb_categorias`. A cláusula `HAVING` serviu para filtrar os resultados e mostrar apenas as categorias que têm a maior soma de preços, utilizando a subconsulta `(SELECT MAX(soma_preços) FROM (SELECT SUM(preço) as soma_preços FROM produtos GROUP BY categoria) as somas)` para obter o valor máximo da soma de preços em todas as categorias. Por fim, a cláusula HAVING então comparou a soma de preços de cada categoria com o valor máximo obtido na subconsulta.

### Referências
<a href="https://youtube.com/playlist?list=PLucm8g_ezqNoAkYKXN_zWupyH6hQCAwxY">Playlist sobre PostgreSQL do canal Bóson Treinamentos</a>

### Créditos da resolução da pergunta 2
<a href="https://github.com/geremiasslima">Geremias Lima</a>
