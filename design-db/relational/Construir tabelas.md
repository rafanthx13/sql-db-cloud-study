1. Decida se será camelCsse ou senak_case
2. Toda tabela temque ter a chave primária como 'id', nunca 'tabela_id', isso guarda apaenas para FK
3. Jamais duplique o nome da tabela como atributo, pois ficará terrível no FrontEnd (city.city, product. product). Mude para: (city.name, product.desc)
