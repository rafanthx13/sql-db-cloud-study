

## Thor

```sql
SELECT idCliente, NomeCliente, Contato_Email,
                        Endereco_Logradouro, Endereco_Numero,
                        Endereco_Complemento, Endereco_Cidade,
                        Endereco_Estado,
                        idt_retorno_audicon, nom_categoria_cliente,
                        nom_tipo_documento, num_documento,
                        nom_cliente, nom_fantasia_cliente,
                        nom_logradouro_endereco, num_cep_endereco,
                        nom_bairro_endereco, nom_cidade_endereco,
                        sgl_uf_endereco, num_inscricao_estadual, 
                        sgl_uf_inscricao_estadual,
                        DATE_FORMAT(dta_ult_alteracao,'%d/%m/%Y %H:%i:%s') as dta_ult_alteracao
                 FROM adia_eai.adia_cliente_odin
                 LEFT JOIN adia_retorno_audicon a ON IdCliente = idt_cliente_odin
                 WHERE DocumentoCliente = :cliente
```



```
SELECT fat.idt_docto_fatura, 
                       fat.cod_fatura_odin, 
                       count(*) qtd_item, 
                       sum(fat.valoritem) val_fatura, 
                       (sum(fat.valoritem) - COALESCE( sum(nota.val_nota_credito), 0)) val_pendente,
                       if(count(nfe.idt_fatura) > 0, 's', 'n')  as checkNfe
                FROM adia_eai.adia_ordem_odin fat
                LEFT JOIN adia_eai.adia_notacredito_odin nota 
                  ON fat.cod_fatura_odin = nota.cod_fatura_odin
                  AND nota.ind_credito_contestacao = 'S'
                LEFT JOIN adia_controle_nfe nfe ON nfe.idt_fatura = fat.cod_fatura_odin
                WHERE tipodaordem = 'Billing Order'
                  AND fat.cod_fatura_odin = :fatura 
                GROUP BY idt_docto_fatura, fat.cod_fatura_odin
```



```
SELECT
                    cli.IdCliente AS id,
                    cli.NomeCliente AS nome,
                    cli.Endereco_Cidade AS cidade,
                    cli.Endereco_Estado AS uf,
                    mi.nome_municipio AS cidade_ibge,
                    mi.nome_UF AS uf_ibge,
                    'adia_cliente_odin' AS tabela
                FROM adia_cliente_odin cli
                LEFT JOIN adia_municipios_ibge mi
                ON mi.nome_municipio LIKE cli.Endereco_Cidade
                WHERE cli.IdCliente <> ''
                AND (cli.Endereco_Cidade NOT IN (SELECT Nome_Municipio FROM adia_municipios_ibge)
                    OR cli.Endereco_Estado NOT IN (SELECT Nome_UF FROM adia_municipios_ibge))
                UNION
                SELECT
                    aud.idt_cliente_odin AS id,
                    if(aud.nom_cliente <> '',aud.nom_cliente, c.NomeCliente) AS nome,
                    aud.nom_cidade_endereco AS cidade,
                    aud.sgl_uf_endereco AS uf,
                    mi.nome_municipio AS cidade_ibge,
                    mi.nome_UF AS uf_ibge,
                    'adia_retorno_audicon' AS tabela
                FROM adia_eai.adia_retorno_audicon aud
                JOIN adia_cliente_odin c ON c.IdCliente = aud.idt_cliente_odin
                LEFT JOIN adia_municipios_ibge mi ON mi.nome_municipio LIKE aud.nom_cidade_endereco
                WHERE (aud.nom_cidade_endereco NOT IN (SELECT Nome_Municipio FROM adia_municipios_ibge)
                OR aud.sgl_uf_endereco NOT IN (SELECT Nome_UF FROM adia_municipios_ibge))
                AND aud.sgl_uf_endereco <> ''
                AND aud.sgl_uf_endereco <> '' "
```



```
SELECT
                    cliente.endereco_estado UF,
                    IFNULL(CASE WHEN ordem.NomeVendedor IS NOT NULL AND ordem.NomeVendedor <> '' AND cliente.segmentocliente = 'Corporativo'
                                        THEN (SELECT cr.num_centro_resultado
                                            FROM adia_cr_consultor_vantive cr
                                            WHERE cr.num_documento_consultor = ordem.NomeVendedor LIMIT 1)
                                        WHEN ordem.NomeVendedor IS NOT NULL AND ordem.NomeVendedor <> '' AND cliente.segmentocliente <> 'Corporativo'
                                    THEN (SELECT crl.num_centro_resultado
                                                FROM adia_cr_localidade_cliente crl
                                            WHERE crl.nom_localidade = cliente.endereco_cidade
                                                AND crl.sgl_uf_localidade = cliente.endereco_estado LIMIT 1)
                                        WHEN ordem.NomeVendedor IS NULL OR ordem.NomeVendedor = ''
                                        THEN (SELECT crl.num_centro_resultado
                                            FROM adia_cr_localidade_cliente crl
                                        WHERE crl.nom_localidade = cliente.endereco_cidade
                                            AND crl.sgl_uf_localidade = cliente.endereco_estado LIMIT 1)
                                ELSE '00'
                            END, '00') cod_centro_custos,
                    cliente.NomeCliente,
                    cliente.DocumentoCliente CPF_CNPJ,
                    ordem.cod_fatura_odin NUM_FATURA,
                    -- ordem.cod_fatura_original, --
                    STR_TO_DATE(ordem.Data_criacao, '%Y-%m-%d') AS EMISSAO,
                    STR_TO_DATE(ordem.Data_criacao + INTERVAL 18 DAY,'%Y-%m-%d') AS VENCIMENTO,
                    IF(ISNULL(pagamento.Datadopagamento), IF ( DATEDIFF(CURDATE(), STR_TO_DATE(ordem.Data_criacao + INTERVAL 18 DAY, '%Y-%m-%d')) <= 0
                        , 0
                        , DATEDIFF(CURDATE(), STR_TO_DATE(ordem.Data_criacao + INTERVAL 18 DAY, '%Y-%m-%d'))
                    )
                        , IF ( DATEDIFF(STR_TO_DATE(pagamento.Datadopagamento, '%Y-%m-%d'), STR_TO_DATE(ordem.Data_criacao + INTERVAL 18 DAY, '%Y-%m-%d')) <= 0
                            , 0
                            , DATEDIFF(STR_TO_DATE(pagamento.Datadopagamento, '%Y-%m-%d'), STR_TO_DATE(ordem.Data_criacao + INTERVAL 18 DAY, '%Y-%m-%d'))
                        )
                    ) AS NUM_DIAS_ATRASO,

                    ordem.valorOrdem AS VALOR_FATURA,

                    ((ordem.valorOrdem + IF(notaCreditoOrdem.valorNotaCredito IS NULL,
                        0.00,
                        notaCreditoOrdem.valorNotaCredito)) - (IF(pagamento.valorPagamento IS NULL,
                        0.00,
                        pagamento.valorPagamento) + (IF(notaCredito.valorNotaCredito IS NULL,
                        0.00,
                        notaCredito.valorNotaCredito)))) SALDO_ABERTO,

                    IF(
                        (COALESCE(pagamento.valorPagamento,0) +  (COALESCE( notaCredito.valorNotaCredito,0))) - ordem.valorOrdem > 0,
                        (COALESCE(pagamento.valorPagamento,0) +  (COALESCE( notaCredito.valorNotaCredito,0))) - ordem.valorOrdem, 0
                    ) AS VALOR_CREDITO,

                    IF(
                        (COALESCE(pagamento.valorPagamento,0) +  (COALESCE( notaCredito.valorNotaCredito,0))) - ordem.valorOrdem > 0,
                        (COALESCE(pagamento.valorPagamento,0) + (COALESCE( notaCredito.valorNotaCredito,0))) - ordem.valorOrdem, 0
                    ) AS CREDITO_CLIENTE,

                    IF(ISNULL(pagamento.DatadoPagamento), (ordem.valorOrdem - (
                        COALESCE( IF(notaCreditoOrdem.valorNotaCredito IS NULL, 0.00,notaCreditoOrdem.valorNotaCredito)))), 0) AS VALOR_GRC_GRS   -- incluir o valor da nota de crédito

                FROM
                    adia_cliente_odin cliente,
                    (SELECT
                            oo.idt_item_ordem,
                            oo.idcliente,
                            max(oo.data_criacao) data_criacao,
                            oo.NomeVendedor,
                            oo.cod_fatura_odin AS cod_fatura_original,
                            CASE
                                WHEN correcao.idt_correcaofatura IS NOT NULL THEN correcao.cod_fatura_odin
                                ELSE oo.cod_fatura_odin
                            END AS cod_fatura_odin,
                            SUM(oo.ValorItem) valorOrdem
                    FROM
                        adia_ordem_odin oo
                    LEFT JOIN adia_correcaofatura AS original ON original.cod_fatura_odin = oo.cod_fatura_odin
                    LEFT JOIN adia_correcaofatura AS correcao ON correcao.cod_correcao_fatura = oo.cod_fatura_odin
                    WHERE
                    (oo.osvenda LIKE 'BO%' OR (oo.osvenda LIKE 'SO%' AND oo.TaxaInstalação > 0 ))
                            AND ModalidadedePagamento = 'POS-PAGO'
                            AND oo.cod_fatura_odin <> ''
                            AND original.idt_correcaofatura IS NULL
                            AND (correcao.cancelado = 0
                            OR correcao.idt_correcaofatura IS NULL)
                    GROUP BY oo.cod_fatura_odin) ordem
                        LEFT JOIN
                    (SELECT
                        idt_nota_credito,
                            cod_fatura_odin,
                            ind_credito_contestacao,
                            SUM(val_nota_credito) valorNotaCredito
                    FROM
                        adia_notacredito_odin
                    GROUP BY cod_fatura_odin) notaCredito ON ordem.cod_fatura_odin = notaCredito.cod_fatura_odin
                        LEFT JOIN
                    (SELECT
                        idt_nota_credito,
                        cod_fatura_odin,
                        ind_credito_contestacao,
                        SUM(val_nota_credito) valorNotaCredito
                    FROM
                        adia_notacredito_odin
                    GROUP BY cod_fatura_odin , ind_credito_contestacao
                    HAVING ind_credito_contestacao = 'N') notaCreditoOrdem ON ordem.cod_fatura_odin = notaCreditoOrdem.cod_fatura_odin
                        LEFT JOIN
                    (SELECT
                        id,
                            IdFatura,
                            idt_pagamento_odin,
                            Datadopagamento,
                            SUM(valorTotal) AS valorPagamento
                    FROM
                        (SELECT
                        id,
                            IdFatura,
                            idt_pagamento_odin,
                            Datadopagamento,
                            SUM(pag.ValorItem) valorPagamento,
                            ValordoPagamento AS valorTotal
                    FROM
                        adia_pagamento_odin pag
                    WHERE
                        ModalidadedePagamento = 'POS-PAGO'
                    GROUP BY IdFatura , idt_pagamento_odin) subpagamento
                    GROUP BY idfatura) pagamento ON ordem.cod_fatura_odin = pagamento.idfatura
                WHERE
                    ordem.idcliente = cliente.IdCliente
                        AND DATE(NOW()) > ordem.Data_criacao + INTERVAL 18 DAY
                        AND ((ordem.valorOrdem + IF(notaCreditoOrdem.valorNotaCredito IS NULL,
                        0.00,
                        notaCreditoOrdem.valorNotaCredito)) - (IF(pagamento.valorPagamento IS NULL,
                        0.00,
                        pagamento.valorPagamento) + (IF(notaCredito.valorNotaCredito IS NULL,
                        0.00,
                        notaCredito.valorNotaCredito)))) > 0.00
                ORDER BY VENCIMENTO ASC
```



```
SELECT
                 pagamento.idprodutocontratado num_contrato
                 , mi.municipio cod_munibge_cliente
                 , mi.nome_municipio nom_cidade_cliente
                 , pagamento.idcliente idt_cliente_odin
                 , c.nomecliente nom_cliente
                 , c.documentocliente num_docto_cliente
                 , pagamento.idfatura cod_fatura_odin
                 , pagamento.data_criacao dta_criacao_item
                 , (date_format(ordem.data_criacao, '%Y-%m-%d') + interval 10 day) dta_vencimento
                 , pagamento.datadopagamento dta_pagamento_item
                 , 'BAIXA' nom_tipo_baixa
                 , sum(ordem.valoritem) val_bruto_fatura
                 , sum(ordem.valoritem) - sum(ordem.imposto_valorpis) - sum(ordem.imposto_valorcofins) - sum(ordem.imposto_valoriss) val_liquido_fatura
                 , c.substituicaointerna nom_tipo_retencao
                 , sum(pagamento.valoritem) val_baixa_fatura, case pagamento.formadepagamento when 'CARTÃO DE CRÉDITO' then round(sum(pagamento.valoritem) * val_perc_taxa_adm, 2) else 0 end val_tarifa
                 , 'BCO 033 AG 3342 C/C 130011448' dsc_conta_receb
                 , pagamento.datadearrecadacao dta_prevista_credito
                 , 1 cod_tipo_pagto, 'NORMAL' nom_tipo_pagto
                 , case pagamento.formadepagamento when 'Boleto' then 1 when 'Cartão de Crédito' then 2 end cod_carteira_pagamento
                 , pagamento.formadepagamento nom_carteira_pagamento
                 , pagamento.bandeiracartao nom_bandeira_cartao
                 , pagamento.modalidadedepagamento nom_modalidade_pagto
            FROM
                (
                    SELECT
                        idfatura,
                        idcliente,
                        idprodutocontratado,
                        DATE_FORMAT(MIN(data_criacao), '%Y-%m-%d') data_criacao,
                        datadopagamento,
                        formadepagamento,
                        bandeiracartao,
                        datadearrecadacao,
                        modalidadedepagamento,
                        SUM(valoritem) valoritem
                    FROM
                        adia_pagamento_odin
                    WHERE
                        Datadopagamento BETWEEN :inicial AND :final
                    GROUP BY
                        idfatura,
                        idcliente,
                        idprodutocontratado,
                        datadopagamento,
                        formadepagamento,
                        bandeiracartao,
                        datadearrecadacao,
                        modalidadedepagamento
                ) pagamento
            INNER JOIN adia_cliente_odin c ON c.idcliente = pagamento.idcliente
            INNER JOIN (
                        SELECT

                            oo.idcliente,
                            sum(oo.valoritem) valoritem,
                            sum(oo.imposto_valorpis) imposto_valorpis,
                            sum(oo.imposto_valorcofins) imposto_valorcofins,
                            sum(oo.imposto_valoriss) imposto_valoriss,
                            MIN(oo.data_criacao) data_criacao,
                            CASE
                                WHEN correcao.idt_correcaofatura IS NOT NULL THEN correcao.cod_fatura_odin
                                ELSE oo.cod_fatura_odin
                            END AS cod_fatura_odin,
                            SUM(oo.ValorItem) valorOrdem
                        FROM
                            adia_ordem_odin oo
                        LEFT JOIN
                            adia_correcaofatura AS original ON original.cod_fatura_odin = oo.cod_fatura_odin
                        LEFT JOIN
                            adia_correcaofatura AS correcao ON correcao.cod_correcao_fatura = oo.cod_fatura_odin
                        WHERE
                            ((oo.modalidadedepagamento = 'POS-PAGO' AND   oo.tipodaordem IN ('Cancellation Order', 'Billing Order'))
                            OR  (oo.modalidadedepagamento = 'PRE-PAGO' AND   oo.tipodaordem IN ('Change Order', 'Sales Order', 'Renewal Order')))
                            AND oo.cod_fatura_odin <> ''
                            AND original.idt_correcaofatura IS NULL
                            AND (correcao.cancelado = 0 OR correcao.idt_correcaofatura IS NULL)
                        GROUP BY oo.cod_fatura_odin, oo.idcliente) ordem
            ON ordem.cod_fatura_odin = pagamento.idfatura AND ordem.idcliente = pagamento.idcliente

            LEFT JOIN adia_municipios_ibge mi
                ON  mi.nome_municipio = c.endereco_cidade
                AND  mi.nome_uf = c.endereco_estado
            LEFT JOIN (SELECT regra, val_perc_taxa_adm FROM adia_acordo_financeiro WHERE tipo = 'CARTÃO DE CRÉDITO') af
                ON af.regra = pagamento.bandeiracartao

            GROUP BY pagamento.idprodutocontratado
                 , mi.municipio, mi.nome_municipio
                 , pagamento.idcliente, c.nomecliente, c.documentocliente
                 , (date_format(ordem.data_criacao, '%Y-%m-%d') + interval 10 day)
                 , pagamento.datadopagamento, c.substituicaointerna
                 , datadearrecadacao
                 , pagamento.formadepagamento
                 , pagamento.bandeiracartao, pagamento.modalidadedepagamento

            UNION ALL

            SELECT nc.cod_contrato_odin num_contrato
                 , mi.municipio cod_munibge_cliente
                 , mi.nome_municipio nom_cidade_cliente
                 , nc.idt_cliente_odin
                 , c.nomecliente nom_cliente
                 , c.documentocliente num_docto_cliente
                 , nc.cod_fatura_odin
                 , nc.dta_geracao_nota dta_criacao_item
                 , NULL dta_vencimento
                 , nc.dta_aplicacao_nota dta_pagamento_item, 'NOTA DE CREDITO' nom_tipo_baixa, o.valoritem val_bruto_fatura
                 , o.valoritem - o.imposto_valorpis - o.imposto_valorcofins - o.imposto_valoriss val_liquido_fatura
                 , c.substituicaointerna nom_tipo_retencao
                 , val_nota_credito val_baixa_fatura
                 , 0 val_tarifa
                 , NULL dsc_conta_receb, NULL dta_prevista_credito
                 , 1 cod_tipo_pagto, 'NORMAL' nom_tipo_pagto
                 , 3 cod_carteira_pagamento
                 , 'Baixa Manual' nom_carteira_pagamento
                 , NULL nom_bandeira_cartao, o.modalidadedepagamento nom_modalidade_pagto
            FROM adia_notacredito_odin nc
            INNER JOIN adia_cliente_odin c
                ON c.idcliente = nc.idt_cliente_odin
            INNER JOIN (
                        SELECT
                            oo.modalidadedepagamento
                            , oo.idcliente
                            , sum(oo.valoritem) valoritem
                            , sum(oo.imposto_valorpis) imposto_valorpis
                            , sum(oo.imposto_valorcofins) imposto_valorcofins
                            , sum(oo.imposto_valoriss) imposto_valoriss
                            , min(oo.data_criacao) data_criacao
                            ,
                            CASE
                                WHEN correcao.idt_correcaofatura IS NOT NULL THEN correcao.cod_fatura_odin
                                ELSE oo.cod_fatura_odin
                            END AS cod_fatura_odin,
                            SUM(oo.ValorItem) valorOrdem
                        FROM
                            adia_ordem_odin oo
                        LEFT JOIN
                            adia_correcaofatura AS original ON original.cod_fatura_odin = oo.cod_fatura_odin
                        LEFT JOIN
                            adia_correcaofatura AS correcao ON correcao.cod_correcao_fatura = oo.cod_fatura_odin
                        WHERE
                            ((oo.modalidadedepagamento = 'POS-PAGO' AND   oo.tipodaordem IN ('Cancellation Order', 'Billing Order'))
                            OR  (oo.modalidadedepagamento = 'PRE-PAGO' AND   oo.tipodaordem IN ('Change Order', 'Sales Order', 'Renewal Order')))
                            AND oo.cod_fatura_odin <> ''
                            AND original.idt_correcaofatura IS NULL
                            AND (correcao.cancelado = 0 OR correcao.idt_correcaofatura IS NULL)
                        GROUP BY oo.cod_fatura_odin, oo.idcliente) o
            ON o.cod_fatura_odin = nc.cod_fatura_odin
            AND o.idcliente = nc.idt_cliente_odin
            LEFT JOIN adia_municipios_ibge mi
            ON  mi.nome_municipio = c.endereco_cidade
            AND  mi.nome_uf = c.endereco_estado
            WHERE ind_credito_contestacao = 'S'
            AND nc.dta_aplicacao_nota BETWEEN :inicial AND :final
```



```
SELECT 
        ife.*,
        c.SegmentoCliente,
        c.NomeCliente,
        IFNULL(CASE
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente = 'Corporativo'
                    THEN
                        (SELECT 
                                cr.num_centro_resultado
                            FROM
                                adia_cr_consultor_vantive cr
                            WHERE
                                cr.num_documento_consultor = ife.nom_vendedor
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente <> 'Corporativo'
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NULL
                            OR ife.nom_vendedor = ''
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    ELSE '00'
                END,
                '00') cod_centro_custos,
        CASE
            WHEN
                ife.val_iss IS NULL
                    AND ife.val_pis IS NULL
                    AND ife.val_cofins IS NULL
            THEN
                'Sem Imposto'
            WHEN
                ife.val_iss IS NOT NULL
                    AND ife.val_pis IS NOT NULL
                    AND ife.val_cofins IS NOT NULL
            THEN
                'Imposto Federal e Municipal'
            WHEN
                ife.val_iss IS NOT NULL
                    AND ife.val_pis IS NULL
                    AND ife.val_cofins IS NULL
            THEN
                'Imposto Municipal'
            ELSE 'Imposto Federal'
        END tipo_imposto,
        IFNULL(ife.val_iss + ife.val_pis + ife.val_cofins,
                0) valor_retido,
        '' AS DATA_CONTABILIZACAO_LANCAMENTO,
        DATE_FORMAT(ife.dta_processamento_nfSe, '%d/%m/%Y') AS DATA_CRIACAO_LANCAMENTO,
        CASE
            WHEN ife.ind_tipo_chave = 'ORDM' THEN 'FATURA'
            ELSE 'PAGAMENTO'
        END CATEGORIA_LANCAMENTO,
        ife.cod_conta_contabil AS CONTA_CONTABIL,
        CASE
            WHEN
                ife.nom_vendedor IS NOT NULL
                    AND ife.nom_vendedor <> ''
                    AND c.segmentocliente = 'Corporativo'
            THEN
                (SELECT 
                        cr.num_centro_resultado
                    FROM
                        adia_cr_consultor_vantive cr
                    WHERE
                        cr.num_documento_consultor = ife.nom_vendedor
                    LIMIT 1)
            WHEN
                ife.nom_vendedor IS NOT NULL
                    AND ife.nom_vendedor <> ''
                    AND c.segmentocliente <> 'Corporativo'
            THEN
                (SELECT 
                        crl.num_centro_resultado
                    FROM
                        adia_cr_localidade_cliente crl
                    WHERE
                        crl.nom_localidade = c.endereco_cidade
                            AND crl.sgl_uf_localidade = c.endereco_estado
                    LIMIT 1)
            WHEN
                ife.nom_vendedor IS NULL
                    OR ife.nom_vendedor = ''
            THEN
                (SELECT 
                        crl.num_centro_resultado
                    FROM
                        adia_cr_localidade_cliente crl
                    WHERE
                        crl.nom_localidade = c.endereco_cidade
                            AND crl.sgl_uf_localidade = c.endereco_estado
                    LIMIT 1)
            ELSE '00'
        END AS CENTRO_RESULTADO,
        CASE LEFT((SELECT 
                    c.num_conta_contabil
                FROM
                    adia_configuracao_contabil c
                WHERE
                    c.nom_categoria_lancto = 'Ajuste'
                        AND c.nom_tipo_evento = 'CONTESTACAO'
                        AND c.nom_sinal_lancamento = 'Debito'),
            1)
            WHEN (1 OR 2) THEN LPAD('0', 12, '0')
            ELSE ife.cod_sku_odin
        END AS PRODUTO,
        
        '0,00' AS VALOR_AJUSTE_IMPOSTOS,
        DATE_FORMAT(CURDATE(), '%d/%m/%Y') AS DATA_CRIACAO,
        CONCAT('016',
                CAST(DATE_FORMAT(NOW(), '%Y%m%d%H%i') AS CHAR)) AS IDENTIFICADOR_LOTE,
        DATE_FORMAT(ife.dta_prevista_credito, '%d/%m/%Y') AS DATA_CREDITO
    FROM
        vw_itemfatura_nfe ife
            INNER JOIN
        adia_cliente_odin c ON ife.idt_cliente_odin = c.idcliente
    WHERE
        dta_criacao_item >= STR_TO_DATE(:start, '%Y%m%d')
            AND dta_criacao_item < STR_TO_DATE(:end, '%Y%m%d')
            
            
            UNION
            
            
            
            SELECT 
        *
    FROM
        (SELECT 
            ife.*,
                c.SegmentoCliente,
                c.NomeCliente,
                IFNULL(CASE
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente = 'Corporativo'
                    THEN
                        (SELECT 
                                cr.num_centro_resultado
                            FROM
                                adia_cr_consultor_vantive cr
                            WHERE
                                cr.num_documento_consultor = ife.nom_vendedor
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente <> 'Corporativo'
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NULL
                            OR ife.nom_vendedor = ''
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    ELSE '00'
                END, '00') cod_centro_custos,
                CASE
                    WHEN
                        ife.val_iss IS NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Sem Imposto'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NOT NULL
                            AND ife.val_cofins IS NOT NULL
                    THEN
                        'Imposto Federal e Municipal'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Imposto Municipal'
                    ELSE 'Imposto Federal'
                END tipo_imposto,
                IFNULL(ife.val_iss + ife.val_pis + ife.val_cofins, 0) valor_retido,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CONTABILIZACAO_LANCAMENTO,
                DATE_FORMAT(ife.dta_processamento_nfSe, '%d/%m/%Y') AS DATA_CRIACAO_LANCAMENTO,
                'IMPOSTO_PIS' AS CATEGORIA_LANCAMENTO,
                (SELECT 
                        c.num_conta_contabil
                    FROM
                        adia_configuracao_contabil c
                    WHERE
                        c.nom_tipo_evento = 'IMPOSTO_PIS'
                            AND c.nom_sinal_lancamento = 'Debito'
                            AND c.nom_evento_adia = 'FATURAMENTO_POS') AS CONTA_CONTABIL,
               CASE
                        WHEN
                            ife.nom_vendedor IS NOT NULL
                                AND ife.nom_vendedor <> ''
                                AND c.segmentocliente = 'Corporativo'
                        THEN
                            (SELECT 
                                    cr.num_centro_resultado
                                FROM
                                    adia_cr_consultor_vantive cr
                                WHERE
                                    cr.num_documento_consultor = ife.nom_vendedor
                                LIMIT 1)
                        WHEN
                            ife.nom_vendedor IS NOT NULL
                                AND ife.nom_vendedor <> ''
                                AND c.segmentocliente <> 'Corporativo'
                        THEN
                            (SELECT 
                                    crl.num_centro_resultado
                                FROM
                                    adia_cr_localidade_cliente crl
                                WHERE
                                    crl.nom_localidade = c.endereco_cidade
                                        AND crl.sgl_uf_localidade = c.endereco_estado
                                LIMIT 1)
                        WHEN
                            ife.nom_vendedor IS NULL
                                OR ife.nom_vendedor = ''
                        THEN
                            (SELECT 
                                    crl.num_centro_resultado
                                FROM
                                    adia_cr_localidade_cliente crl
                                WHERE
                                    crl.nom_localidade = c.endereco_cidade
                                        AND crl.sgl_uf_localidade = c.endereco_estado
                                LIMIT 1)
                        ELSE '00'
                   
                END AS CENTRO_RESULTADO,
                ife.cod_sku_odin AS PRODUTO,
                CAST(REPLACE(ife.val_pis, '.', ',') AS CHAR) AS VALOR_AJUSTE_IMPOSTOS,
                DATE_FORMAT(CURDATE(), '%d/%m/%Y') AS DATA_CRIACAO,
                CONCAT('016', CAST(DATE_FORMAT(NOW(), '%Y%m%d%H%i') AS CHAR)) AS IDENTIFICADOR_LOTE,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CREDITO
        FROM
            vw_itemfatura_nfe ife
        INNER JOIN adia_cliente_odin c ON ife.idt_cliente_odin = c.idcliente
        WHERE
            nom_modalidade_pagto = 'POS-PAGO'
                AND dta_emissao_rps >= STR_TO_DATE(:start, '%Y%m%d')
                AND dta_emissao_rps < STR_TO_DATE(:end, '%Y%m%d') 
                UNION ALL 
                
                -- QUERY USADA PARA CONSULTA DE AJUSTES DE POS PAGO PARA CREDITO
    SELECT 
        ife.*,
        c.SegmentoCliente,
        c.NomeCliente,
        IFNULL(CASE
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente = 'Corporativo'
                    THEN
                        (SELECT 
                                cr.num_centro_resultado
                            FROM
                                adia_cr_consultor_vantive cr
                            WHERE
                                cr.num_documento_consultor = ife.nom_vendedor
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente <> 'Corporativo'
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NULL
                            OR ife.nom_vendedor = ''
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    ELSE '00'
                END,
                '00') cod_centro_custos,
        CASE
            WHEN
                ife.val_iss IS NULL
                    AND ife.val_pis IS NULL
                    AND ife.val_cofins IS NULL
            THEN
                'Sem Imposto'
            WHEN
                ife.val_iss IS NOT NULL
                    AND ife.val_pis IS NOT NULL
                    AND ife.val_cofins IS NOT NULL
            THEN
                'Imposto Federal e Municipal'
            WHEN
                ife.val_iss IS NOT NULL
                    AND ife.val_pis IS NULL
                    AND ife.val_cofins IS NULL
            THEN
                'Imposto Municipal'
            ELSE 'Imposto Federal'
        END tipo_imposto,
        IFNULL(ife.val_iss + ife.val_pis + ife.val_cofins,
                0) valor_retido,
        DATE_FORMAT(nc.dta_aplicacao_nota, '%d/%m/%Y') AS DATA_CONTABILIZACAO_LANCAMENTO,
        DATE_FORMAT(ife.dta_processamento_nfSe, '%d/%m/%Y') AS DATA_CRIACAO_LANCAMENTO,
        (SELECT DISTINCT
                UPPER(c.nom_categoria_lancto)
            FROM
                adia_configuracao_contabil c
            WHERE
                c.nom_categoria_lancto = 'Ajuste'
                    AND c.nom_tipo_evento = 'CONTESTACAO'
                    AND c.nom_sinal_lancamento = 'Credito') AS CATEGORIA_LANCAMENTO,
        (SELECT 
                c.num_conta_contabil
            FROM
                adia_configuracao_contabil c
            WHERE
                c.nom_categoria_lancto = 'Ajuste'
                    AND c.nom_tipo_evento = 'CONTESTACAO'
                    AND c.nom_sinal_lancamento = 'Credito') AS CONTA_CONTABIL,
        CASE
                                WHEN
                                    ife.nom_vendedor IS NOT NULL
                                        AND ife.nom_vendedor <> ''
                                        AND c.segmentocliente = 'Corporativo'
                                THEN
                                    (SELECT 
                                            cr.num_centro_resultado
                                        FROM
                                            adia_cr_consultor_vantive cr
                                        WHERE
                                            cr.num_documento_consultor = ife.nom_vendedor
                                        LIMIT 1)
                                WHEN
                                    ife.nom_vendedor IS NOT NULL
                                        AND ife.nom_vendedor <> ''
                                        AND c.segmentocliente <> 'Corporativo'
                                THEN
                                    (SELECT 
                                            crl.num_centro_resultado
                                        FROM
                                            adia_cr_localidade_cliente crl
                                        WHERE
                                            crl.nom_localidade = c.endereco_cidade
                                                AND crl.sgl_uf_localidade = c.endereco_estado
                                        LIMIT 1)
                                WHEN
                                    ife.nom_vendedor IS NULL
                                        OR ife.nom_vendedor = ''
                                THEN
                                    (SELECT 
                                            crl.num_centro_resultado
                                        FROM
                                            adia_cr_localidade_cliente crl
                                        WHERE
                                            crl.nom_localidade = c.endereco_cidade
                                                AND crl.sgl_uf_localidade = c.endereco_estado
                                        LIMIT 1)
                                ELSE '00'
                            
        END AS CENTRO_RESULTADO,
        ife.cod_sku_odin AS PRODUTO,
         
        CAST(REPLACE(nc.val_nota_credito, '.', ',') AS CHAR) AS VALOR_AJUSTE_IMPOSTOS,
        DATE_FORMAT(CURDATE(), '%d/%m/%Y') AS DATA_CRIACAO,
        CONCAT('016',
                CAST(DATE_FORMAT(NOW(), '%Y%m%d%H%i') AS CHAR)) AS IDENTIFICADOR_LOTE,
        DATE_FORMAT(nc.dta_aplicacao_nota, '%d/%m/%Y') AS DATA_CREDITO
    FROM
        vw_itemfatura_nfe ife
            INNER JOIN
        adia_cliente_odin c ON ife.idt_cliente_odin = c.idcliente
            INNER JOIN
        adia_notacredito_odin nc ON nc.idt_cliente_odin = c.idcliente
    WHERE
        nom_modalidade_pagto = 'POS-PAGO'
            AND ind_credito_contestacao = 'S'
            AND dta_aplicacao_nota >= STR_TO_DATE(:start, '%Y%m%d')
            AND dta_aplicacao_nota < STR_TO_DATE(:end, '%Y%m%d')
    GROUP BY idt_nota_credito 
                
                UNION ALL
                SELECT 
            ife.*,
                c.SegmentoCliente,
                c.NomeCliente,
                IFNULL(CASE
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente = 'Corporativo'
                    THEN
                        (SELECT 
                                cr.num_centro_resultado
                            FROM
                                adia_cr_consultor_vantive cr
                            WHERE
                                cr.num_documento_consultor = ife.nom_vendedor
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente <> 'Corporativo'
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NULL
                            OR ife.nom_vendedor = ''
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    ELSE '00'
                END, '00') cod_centro_custos,
                CASE
                    WHEN
                        ife.val_iss IS NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Sem Imposto'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NOT NULL
                            AND ife.val_cofins IS NOT NULL
                    THEN
                        'Imposto Federal e Municipal'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Imposto Municipal'
                    ELSE 'Imposto Federal'
                END tipo_imposto,
                IFNULL(ife.val_iss + ife.val_pis + ife.val_cofins, 0) valor_retido,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CONTABILIZACAO_LANCAMENTO,
                DATE_FORMAT(ife.dta_processamento_nfSe, '%d/%m/%Y') AS DATA_CRIACAO_LANCAMENTO,
                'IMPOSTO_COFINS' AS CATEGORIA_LANCAMENTO,
                (SELECT 
                        c.num_conta_contabil
                    FROM
                        adia_configuracao_contabil c
                    WHERE
                        c.nom_tipo_evento = 'IMPOSTO_COFINS'
                            AND c.nom_sinal_lancamento = 'Debito'
                            AND c.nom_evento_adia = 'FATURAMENTO_POS') AS CONTA_CONTABIL,
               CASE
                        WHEN
                            ife.nom_vendedor IS NOT NULL
                                AND ife.nom_vendedor <> ''
                                AND c.segmentocliente = 'Corporativo'
                        THEN
                            (SELECT 
                                    cr.num_centro_resultado
                                FROM
                                    adia_cr_consultor_vantive cr
                                WHERE
                                    cr.num_documento_consultor = ife.nom_vendedor
                                LIMIT 1)
                        WHEN
                            ife.nom_vendedor IS NOT NULL
                                AND ife.nom_vendedor <> ''
                                AND c.segmentocliente <> 'Corporativo'
                        THEN
                            (SELECT 
                                    crl.num_centro_resultado
                                FROM
                                    adia_cr_localidade_cliente crl
                                WHERE
                                    crl.nom_localidade = c.endereco_cidade
                                        AND crl.sgl_uf_localidade = c.endereco_estado
                                LIMIT 1)
                        WHEN
                            ife.nom_vendedor IS NULL
                                OR ife.nom_vendedor = ''
                        THEN
                            (SELECT 
                                    crl.num_centro_resultado
                                FROM
                                    adia_cr_localidade_cliente crl
                                WHERE
                                    crl.nom_localidade = c.endereco_cidade
                                        AND crl.sgl_uf_localidade = c.endereco_estado
                                LIMIT 1)
                        ELSE '00'
                   
                END AS CENTRO_RESULTADO,
                ife.cod_sku_odin AS PRODUTO,
                CAST(REPLACE(ife.val_pis, '.', ',') AS CHAR) AS VALOR_AJUSTE_IMPOSTOS,
                DATE_FORMAT(CURDATE(), '%d/%m/%Y') AS DATA_CRIACAO,
                CONCAT('016', CAST(DATE_FORMAT(NOW(), '%Y%m%d%H%i') AS CHAR)) AS IDENTIFICADOR_LOTE,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CREDITO
        FROM
            vw_itemfatura_nfe ife
        INNER JOIN adia_cliente_odin c ON ife.idt_cliente_odin = c.idcliente
        WHERE
            nom_modalidade_pagto = 'POS-PAGO'
                AND dta_emissao_rps >= STR_TO_DATE(:start, '%Y%m%d')
                AND dta_emissao_rps < STR_TO_DATE(:end, '%Y%m%d') UNION ALL SELECT 
            ife.*,
                c.SegmentoCliente,
                c.NomeCliente,
                IFNULL(CASE
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente = 'Corporativo'
                    THEN
                        (SELECT 
                                cr.num_centro_resultado
                            FROM
                                adia_cr_consultor_vantive cr
                            WHERE
                                cr.num_documento_consultor = ife.nom_vendedor
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente <> 'Corporativo'
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NULL
                            OR ife.nom_vendedor = ''
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    ELSE '00'
                END, '00') cod_centro_custos,
                CASE
                    WHEN
                        ife.val_iss IS NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Sem Imposto'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NOT NULL
                            AND ife.val_cofins IS NOT NULL
                    THEN
                        'Imposto Federal e Municipal'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Imposto Municipal'
                    ELSE 'Imposto Federal'
                END tipo_imposto,
                IFNULL(ife.val_iss + ife.val_pis + ife.val_cofins, 0) valor_retido,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CONTABILIZACAO_LANCAMENTO,
                DATE_FORMAT(ife.dta_processamento_nfSe, '%d/%m/%Y') AS DATA_CRIACAO_LANCAMENTO,
                'IMPOSTO_ISS' AS CATEGORIA_LANCAMENTO,
                (SELECT 
                        c.num_conta_contabil
                    FROM
                        adia_configuracao_contabil c
                    WHERE
                        c.nom_tipo_evento = 'IMPOSTO_ISS'
                            AND c.nom_sinal_lancamento = 'Debito'
                            AND c.nom_evento_adia = 'FATURAMENTO_POS') AS CONTA_CONTABIL,
                CASE
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente = 'Corporativo'
                    THEN
                        (SELECT 
                                cr.num_centro_resultado
                            FROM
                                adia_cr_consultor_vantive cr
                            WHERE
                                cr.num_documento_consultor = ife.nom_vendedor
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente <> 'Corporativo'
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NULL
                            OR ife.nom_vendedor = ''
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    ELSE '00'
                END AS CENTRO_RESULTADO,
                ife.cod_sku_odin AS PRODUTO,
                CAST(REPLACE(ife.val_pis, '.', ',') AS CHAR) AS VALOR_AJUSTE_IMPOSTOS,
                DATE_FORMAT(CURDATE(), '%d/%m/%Y') AS DATA_CRIACAO,
                CONCAT('016', CAST(DATE_FORMAT(NOW(), '%Y%m%d%H%i') AS CHAR)) AS IDENTIFICADOR_LOTE,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CREDITO
        FROM
            vw_itemfatura_nfe ife
        INNER JOIN adia_cliente_odin c ON ife.idt_cliente_odin = c.idcliente
        WHERE
            nom_modalidade_pagto = 'POS-PAGO'
                AND dta_emissao_rps >= STR_TO_DATE(:start, '%Y%m%d')
                AND dta_emissao_rps < STR_TO_DATE(:end, '%Y%m%d') UNION ALL SELECT 
            ife.*,
                c.SegmentoCliente,
                c.NomeCliente,
                IFNULL(CASE
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente = 'Corporativo'
                    THEN
                        (SELECT 
                                cr.num_centro_resultado
                            FROM
                                adia_cr_consultor_vantive cr
                            WHERE
                                cr.num_documento_consultor = ife.nom_vendedor
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NOT NULL
                            AND ife.nom_vendedor <> ''
                            AND c.segmentocliente <> 'Corporativo'
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    WHEN
                        ife.nom_vendedor IS NULL
                            OR ife.nom_vendedor = ''
                    THEN
                        (SELECT 
                                crl.num_centro_resultado
                            FROM
                                adia_cr_localidade_cliente crl
                            WHERE
                                crl.nom_localidade = c.endereco_cidade
                                    AND crl.sgl_uf_localidade = c.endereco_estado
                            LIMIT 1)
                    ELSE '00'
                END, '00') cod_centro_custos,
                CASE
                    WHEN
                        ife.val_iss IS NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Sem Imposto'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NOT NULL
                            AND ife.val_cofins IS NOT NULL
                    THEN
                        'Imposto Federal e Municipal'
                    WHEN
                        ife.val_iss IS NOT NULL
                            AND ife.val_pis IS NULL
                            AND ife.val_cofins IS NULL
                    THEN
                        'Imposto Municipal'
                    ELSE 'Imposto Federal'
                END tipo_imposto,
                IFNULL(ife.val_iss + ife.val_pis + ife.val_cofins, 0) valor_retido,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CONTABILIZACAO_LANCAMENTO,
                DATE_FORMAT(ife.dta_processamento_nfSe, '%d/%m/%Y') AS DATA_CRIACAO_LANCAMENTO,
                'RETENCAO_ISS' AS CATEGORIA_LANCAMENTO,
                (SELECT 
                        c.num_conta_contabil
                    FROM
                        adia_configuracao_contabil c
                    WHERE
                        c.nom_tipo_evento = 'RETENCAO_ISS'
                            AND c.nom_sinal_lancamento = 'Debito'
                            AND c.nom_evento_adia = 'FATURAMENTO_POS') AS CONTA_CONTABIL,
                '000000000' AS CENTRO_RESULTADO,
                '000000000' AS PRODUTO,
                CAST(REPLACE(ife.val_iss, '.', ',') AS CHAR) AS VALOR_AJUSTE_IMPOSTOS,
                DATE_FORMAT(CURDATE(), '%d/%m/%Y') AS DATA_CRIACAO,
                CONCAT('016', CAST(DATE_FORMAT(NOW(), '%Y%m%d%H%i') AS CHAR)) AS IDENTIFICADOR_LOTE,
                DATE_FORMAT(ife.dta_emissao_rps, '%d/%m/%Y') AS DATA_CREDITO
        FROM
            vw_itemfatura_nfe ife
        INNER JOIN adia_cliente_odin c ON ife.idt_cliente_odin = c.idcliente
        WHERE
            nom_modalidade_pagto = 'POS-PAGO'
                AND dta_emissao_rps >= STR_TO_DATE(:start, '%Y%m%d')
                AND dta_emissao_rps < STR_TO_DATE(:end, '%Y%m%d')
                AND c.SubstituicaoInterna = 'Municipal') sub;
```



```
SELECT
                ife.num_documento_cliente,
                ca.nom_cliente,
                ife.cod_vendedor,
                ife.cod_fatura_odin,
                ife.cod_itemfatura_odin,
                ife.val_item_fatura,
                DATE_FORMAT(ife.dta_criacao_item,
                        '%d/%m/%Y %H:%i:%s') AS dta_criacao_item
            FROM
                (SELECT DISTINCT
                    vw.num_documento_cliente,
                        vw.cod_vendedor,
                        CASE
                            WHEN correcao.idt_correcaofatura IS NOT NULL THEN correcao.cod_fatura_odin
                            ELSE vw.cod_fatura_odin
                        END AS cod_fatura_odin,
                        vw.cod_itemfatura_odin,
                        vw.val_item_fatura,
                        vw.dta_criacao_item,
                        vw.nom_modalidade_pagto,
                        vw.idt_cliente_odin
                FROM
                    vw_itemfatura_nfe AS vw
                LEFT JOIN adia_correcaofatura AS original ON original.cod_fatura_odin = vw.cod_fatura_odin
                LEFT JOIN adia_correcaofatura AS correcao ON correcao.cod_correcao_fatura = vw.cod_fatura_odin
                WHERE
                    original.idt_correcaofatura IS NULL
                        AND (correcao.cancelado = 0
                        OR correcao.idt_correcaofatura IS NULL)) AS ife,
                vw_dw_cliente_adia ca
            WHERE
                ife.nom_modalidade_pagto = 'POS-PAGO'
                    AND ife.cod_vendedor IS NOT NULL
                    AND ife.cod_vendedor <> ''
                    AND ife.idt_cliente_odin = ca.idt_cliente_odin
                    AND dta_criacao_item BETWEEN :start AND :end 
```





```
SELECT * FROM (
                    (
                        SELECT
                            'CLOUD' AS Legacy_System_Ident
                            , pag.cod_fatura_odin AS Orig_System_Ref_Id
                            , '5403' AS CodCidade
                            , '04622116000113' AS CPFCNPJRemetente
                            , 'CTBC MULTIMIDIA DATA NET S/A' AS RazaoSocialRemetente
                            , '18308000' AS InscricaoMunicipalPrestador
                            , 'CTBC MULTIMIDIA DATA NET S/A' AS RazaoSocialPrestador
                            , 'RPS' AS TipoRPS
                            , 'NF' AS SerieRPS
                            , pag.Data_criacao AS DataEmissaoRPS
                            , 'N' AS SituacaoRPS
                            , NULL AS NumeroRPS
                            , '99' AS SeriePrestacao
                            , '0000000' AS InscricaoMunicipalTomador
                            , cli.DOCUMENTOCLIENTE AS CPFCNPJTomador
                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , cli.NomeCliente
                            , IF (audicon.nom_cliente IS NULL OR audicon.nom_cliente = '', cli.NomeCliente , audicon.nom_cliente)
                            ) AS RazaoSocialTomador

                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , IF( cli.endereco_Tipo IS NULL OR cli.endereco_Tipo = '' , '-', cli.endereco_Tipo )
                            , '-'
                            ) AS TipoLogradouroTomador
                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , cli.endereco_Logradouro
                            , IF (audicon.nom_logradouro_endereco IS NULL OR audicon.nom_logradouro_endereco = '', cli.endereco_Logradouro, audicon.nom_logradouro_endereco)
                            ) AS LogradouroTomador
                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , IF( cli.endereco_Numero IS NULL OR cli.endereco_Numero = '', '0', cli.endereco_Numero )
                            , IF (audicon.num_endereco IS NULL OR audicon.num_endereco = '', IF( cli.endereco_Numero IS NULL OR cli.endereco_Numero = '', '0', cli.endereco_Numero ), audicon.num_endereco)
                            ) AS NumeroEnderecoTomador
                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , cli.endereco_Complemento
                            , IF (audicon.dsc_complemento_endereco IS NULL OR audicon.dsc_complemento_endereco='','', audicon.dsc_complemento_endereco)
                            ) AS ComplementoEnderecoTomador
                            , 'Bairro' AS TipoBairroTomador
                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , cli.endereco_Bairro
                            , IF (audicon.nom_bairro_endereco IS NULL OR audicon.nom_bairro_endereco='', cli.endereco_Bairro , audicon.nom_bairro_endereco)
                            ) AS BairroTomador

                            , IFNULL( IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , (SELECT Cod_Mun_SIAFI FROM adia_municipios_siafi WHERE UPPER(UF) = UPPER(cli.Endereco_Estado) AND UPPER(Nome) = UPPER(cli.endereco_Cidade) LIMIT 1)
                            , IF (audicon.nom_cidade_endereco IS NULL OR audicon.nom_cidade_endereco = '', (SELECT Cod_Mun_SIAFI FROM adia_municipios_siafi WHERE UPPER(UF) = UPPER(cli.Endereco_Estado) AND UPPER(Nome) = UPPER(cli.endereco_Cidade) LIMIT 1), (SELECT Cod_Mun_SIAFI FROM adia_municipios_siafi WHERE UPPER(UF) = UPPER(audicon.sgl_uf_endereco) AND UPPER(Nome) = UPPER(audicon.nom_cidade_endereco) LIMIT 1))
                            ), '-') AS CidadeTomador


                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , cli.endereco_Cidade
                            , IF (audicon.nom_cidade_endereco IS NULL OR audicon.nom_cidade_endereco = '', cli.endereco_Cidade, audicon.nom_cidade_endereco)
                            ) AS CidadeTomadorDescricao
                            , IF( cli.CategoriadoCliente
                            = 'Pessoa Física' , cli.endereco_CEP
                            , IF ( audicon.num_cep_endereco IS NULL OR audicon.num_cep_endereco = '' , cli.endereco_CEP, REPLACE(REPLACE(audicon.num_cep_endereco, '.', ''), '-',''))
                            ) AS CEPTomador

                            , cli.contato_Email AS EmailTomador
                            , pag.CodigoAtividadeProduto AS CodigoAtividade
                            , pag.Imposto_AliquotaISS AS AliquotaAtividade
                            , CASE cli.SubstituicaoInterna
                            WHEN 'Municipal' THEN 'R'
                            ELSE 'A'
                            END AS TipoRecolhimento
                            , '5403' AS MunicipioPrestacao
                            , 'UBERLANDIA' AS MunicipioPrestacaoDescricao
                            , 'B' AS Operacao
                            , 'T' AS Tributacao
                            , CONCAT('PRESTACAO DE SERVICOS ' , pag.NomeProduto) AS DescricaoRPS
                            , '00' AS DDDPrestador
                            , '00000000' AS TelefonePrestador
                            , '00' AS DDDTomador
                            , cli.Contato_Telefone AS TelefoneTomador
                            , NULL AS MotCancelamento
                            , NULL AS CpfCnpjIntermediario
                            , pag.cod_itemfatura_odin AS Orig_System_Line_Ref_Id
                            , pag.NomeProduto AS DiscriminacaoServico
                            , pag.QuantidadeProduto AS Quantidade
                            , pag.val_unitario AS ValorUnitario
                            , pag.ValorItem AS ValorTotal
                        FROM adia_ordem_odin pag
                        INNER JOIN adia_cliente_odin cli ON cli.idCliente = pag.idCliente
                        LEFT JOIN adia_retorno_audicon audicon ON audicon.idt_cliente_odin = cli.idCliente AND audicon.ind_status_execucao=0
                        WHERE pag.ModalidadedePagamento = 'POS-PAGO'
                            AND pag.TipodaOrdem in ('Billing Order','Debit Memo')
                            AND pag.ValorItem > 0
                            AND pag.Data_criacao >= DATE_SUB(CURDATE(),INTERVAL 90 DAY)
                            AND CONCAT(pag.cod_fatura_odin,'|',pag.cod_itemfatura_odin) not in (SELECT id_Registro
                                FROM adia_controle_execucao_registro_processado
                                WHERE Tipo_Registro = '2.1-FATURAMENTO_NFE'
                                AND Executado_sucesso = 1)
                    )
                    UNION
                    (
                            SELECT
                                'CLOUD' AS Legacy_System_Ident
                                , pag.IdFatura AS Orig_System_Ref_Id
                                , '5403' AS CodCidade
                                , '04622116000113' AS CPFCNPJRemetente
                                , 'CTBC MULTIMIDIA DATA NET S/A' AS RazaoSocialRemetente
                                , '18308000' AS InscricaoMunicipalPrestador
                                , 'CTBC MULTIMIDIA DATA NET S/A' AS RazaoSocialPrestador
                                , 'RPS' AS TipoRPS
                                , 'NF' AS SerieRPS
                                , pag.Data_criacao AS DataEmissaoRPS
                                , 'N' AS SituacaoRPS
                                , NULL AS NumeroRPS
                                , '99' AS SeriePrestacao
                                , '0000000' AS InscricaoMunicipalTomador
                                , cli.DOCUMENTOCLIENTE AS CPFCNPJTomador

                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , cli.NomeCliente
                                    , IF (audicon.nom_cliente IS NULL OR audicon.nom_cliente = '', cli.NomeCliente , audicon.nom_cliente)
                                ) AS RazaoSocialTomador

                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , IF( cli.endereco_Tipo IS NULL OR cli.endereco_Tipo = '' , '-', cli.endereco_Tipo )
                                    , '-'
                                ) AS TipoLogradouroTomador
                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , cli.endereco_Logradouro
                                    , IF (audicon.nom_logradouro_endereco IS NULL OR audicon.nom_logradouro_endereco = '', cli.endereco_Logradouro, audicon.nom_logradouro_endereco)
                                ) AS LogradouroTomador
                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , IF( cli.endereco_Numero IS NULL OR cli.endereco_Numero = '', '0', cli.endereco_Numero )
                                    , IF (audicon.num_endereco IS NULL OR audicon.num_endereco = '', IF( cli.endereco_Numero IS NULL OR cli.endereco_Numero = '', '0', cli.endereco_Numero ), audicon.num_endereco)
                                ) AS NumeroEnderecoTomador
                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , cli.endereco_Complemento
                                    , IF (audicon.dsc_complemento_endereco IS NULL OR audicon.dsc_complemento_endereco='','', audicon.dsc_complemento_endereco)
                                ) AS ComplementoEnderecoTomador
                                , 'Bairro' AS TipoBairroTomador
                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , cli.endereco_Bairro
                                    , IF (audicon.nom_bairro_endereco IS NULL OR audicon.nom_bairro_endereco='', cli.endereco_Bairro , audicon.nom_bairro_endereco)
                                ) AS BairroTomador

                                , IFNULL( IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , (SELECT Cod_Mun_SIAFI FROM adia_municipios_siafi WHERE  UPPER(UF) = UPPER(cli.Endereco_Estado) AND UPPER(Nome) = UPPER(cli.endereco_Cidade) LIMIT 1)
                                    , IF (audicon.nom_cidade_endereco IS NULL OR audicon.nom_cidade_endereco = '', (SELECT Cod_Mun_SIAFI FROM adia_municipios_siafi WHERE  UPPER(UF) = UPPER(cli.Endereco_Estado) AND UPPER(Nome) = UPPER(cli.endereco_Cidade) LIMIT 1), (SELECT Cod_Mun_SIAFI FROM adia_municipios_siafi WHERE  UPPER(UF) = UPPER(audicon.sgl_uf_endereco) AND UPPER(Nome) = UPPER(audicon.nom_cidade_endereco) LIMIT 1))
                                ), '-') AS CidadeTomador

                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , cli.endereco_Cidade
                                    , IF (audicon.nom_cidade_endereco IS NULL OR audicon.nom_cidade_endereco = '', cli.endereco_Cidade, audicon.nom_cidade_endereco)
                                ) AS CidadeTomadorDescricao
                                , IF( cli.CategoriadoCliente
                                    = 'Pessoa Física' , cli.endereco_CEP
                                    , IF ( audicon.num_cep_endereco IS NULL OR audicon.num_cep_endereco = '' , cli.endereco_CEP, REPLACE(REPLACE(audicon.num_cep_endereco, '.', ''), '-',''))
                                ) AS CEPTomador

                                , cli.contato_Email AS EmailTomador
                                , pag.CodigoAtividadeProduto AS CodigoAtividade
                                , pag.Imposto_AliquotaISS AS AliquotaAtividade
                                , CASE cli.SubstituicaoInterna
                                    WHEN 'Municipal' THEN 'R'
                                    ELSE 'A'
                                END AS TipoRecolhimento
                                , '5403' AS MunicipioPrestacao
                                , 'UBERLANDIA' AS MunicipioPrestacaoDescricao
                                , 'B' AS Operacao
                                , 'T' AS Tributacao
                                , CONCAT('PRESTACAO DE SERVICOS ' , pag.NomeProduto) AS DescricaoRPS
                                , '00' AS DDDPrestador
                                , '00000000' AS TelefonePrestador
                                , '00' AS DDDTomador
                                , cli.Contato_Telefone AS TelefoneTomador
                                , NULL AS MotCancelamento
                                , NULL AS CpfCnpjIntermediario
                                , pag.IdItemFatura AS Orig_System_Line_Ref_Id
                                , pag.NomeProduto AS DiscriminacaoServico
                                , pag.QuantidadeProduto AS Quantidade
                                , pag.val_unitario AS ValorUnitario
                                , pag.ValorItem AS ValorTotal
                            FROM adia_pagamento_odin pag
                            INNER JOIN adia_cliente_odin cli ON cli.idCliente = pag.idCliente
                            LEFT JOIN adia_retorno_audicon audicon ON audicon.idt_cliente_odin = cli.idCliente AND audicon.ind_status_execucao=0
                            WHERE
                                pag.ModalidadedePagamento = 'PRE-PAGO'
                                AND pag.Datadopagamento >= DATE_SUB(CURDATE(),INTERVAL 90 DAY)
                                AND CONCAT(pag.IdFatura,'|',pag.IdItemFatura) not in (SELECT id_Registro
                                    FROM adia_controle_execucao_registro_processado
                                    WHERE Tipo_Registro = '2.1-PAGAMENTO_NFE'
                                    AND Executado_sucesso = 1)
                    )
                ) AS t
                ORDER BY Orig_System_Ref_Id ;
```





```
SELECT 
                    desb.faturas, desb.status_cliente, desb.idcliente, desb.data_criacao,
                    CASE
                        WHEN desb.finalizado THEN 'Processo de desbloqueio finalizado'
                        WHEN desb.dias_para_desbloqueio < 0 THEN 'Já está Rebloqueado' 
                        WHEN desb.dias_para_desbloqueio = 0 THEN 'Será Rebloqueado hoje'
                        WHEN desb.dias_para_desbloqueio = 1 THEN concat('Falta ', dias_para_desbloqueio, ' dia')
                        ELSE concat('Faltam ', desb.dias_para_desbloqueio, ' dias')
                    END as dias_para_desbloqueio
                FROM (
                    SELECT 
                        group_concat( concat(  fat.cod_fatura_odin, ' - ', DATE_FORMAT(fat.data_vencimento_original, '%d/%m/%Y')) SEPARATOR '<br>') as faturas,
                        IF(cli.rebloqueado = 1, 'Rebloqueado', 'Liberado') as status_cliente,
                        cli.finalizado,
                        cli.idcliente,
                        cli.data_criacao,
                        datediff(date_add(cli.data_criacao, 
                            INTERVAL (SELECT valor FROM adia_config_kettle WHERE chave = 'ALGAR_DIAS_DESBLOQUEIO_EM_CONFIANCA') DAY),  
                            current_date()) as dias_para_desbloqueio
                    FROM
                        adia_desbloqueio_confianca_fatura fat
                    INNER JOIN 
                        adia_desbloqueio_confianca cli
                        ON cli.id = fat.id_desbloqueio
                    WHERE
                        fat.data_vencimento_original >= STR_TO_DATE(:start_date, '%d/%m/%Y')
                        AND fat.data_vencimento_original < STR_TO_DATE(:final_date, '%d/%m/%Y')
                    GROUP BY
                        cli.id) as desb;
```

