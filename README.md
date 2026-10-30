-- =====================================================
-- PROJETO: DELIVERY CONNECT
-- BANCO DE DADOS: delivery_connect
-- COMPATIBILIDADE: MYSQL / MARIADB
-- =====================================================


-- 1. CRIAR BANCO DE DADOS


CREATE DATABASE IF NOT EXISTS delivery_connect;


USE delivery_connect;




-- =====================================================
-- 2. CRIAR AS TABELAS
-- =====================================================


-- TABELA CLIENTES


CREATE TABLE clientes (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    telefone VARCHAR(20) NOT NULL UNIQUE,
    email VARCHAR(120) NOT NULL UNIQUE,
    endereco VARCHAR(180) NOT NULL,
    criado_em DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;




-- TABELA RESTAURANTES


CREATE TABLE restaurantes (
    id_restaurante INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    categoria VARCHAR(60) NOT NULL,
    telefone VARCHAR(20) NOT NULL UNIQUE,
    endereco VARCHAR(180) NOT NULL,
    ativo BOOLEAN DEFAULT TRUE
) ENGINE=InnoDB;




-- TABELA ENTREGADORES


CREATE TABLE entregadores (
    id_entregador INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    telefone VARCHAR(20) NOT NULL UNIQUE,
    veiculo VARCHAR(30) NOT NULL,
    disponivel BOOLEAN DEFAULT TRUE
) ENGINE=InnoDB;




-- TABELA PRODUTOS


CREATE TABLE produtos (
    id_produto INT AUTO_INCREMENT PRIMARY KEY,
    id_restaurante INT NOT NULL,
    nome VARCHAR(120) NOT NULL,
    descricao VARCHAR(255),
    preco DECIMAL(10,2) NOT NULL,
    disponivel BOOLEAN DEFAULT TRUE,


    CONSTRAINT chk_preco CHECK (preco >= 0),


    FOREIGN KEY (id_restaurante)
    REFERENCES restaurantes(id_restaurante)
) ENGINE=InnoDB;




-- TABELA PEDIDOS


CREATE TABLE pedidos (
    id_pedido INT AUTO_INCREMENT PRIMARY KEY,
    id_cliente INT NOT NULL,
    id_restaurante INT NOT NULL,
    id_entregador INT NULL,
    data_pedido DATETIME DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(25) DEFAULT 'PENDENTE',
    taxa_entrega DECIMAL(10,2) DEFAULT 5.00,
    valor_total DECIMAL(10,2) DEFAULT 0.00,
    observacao VARCHAR(255),


    CONSTRAINT chk_status CHECK (
        status IN (
            'PENDENTE',
            'CONFIRMADO',
            'EM_PREPARACAO',
            'SAIU_PARA_ENTREGA',
            'ENTREGUE',
            'CANCELADO'
        )
    ),


    CONSTRAINT chk_taxa CHECK (taxa_entrega >= 0),
    CONSTRAINT chk_total CHECK (valor_total >= 0),


    FOREIGN KEY (id_cliente)
    REFERENCES clientes(id_cliente),


    FOREIGN KEY (id_restaurante)
    REFERENCES restaurantes(id_restaurante),


    FOREIGN KEY (id_entregador)
    REFERENCES entregadores(id_entregador)
) ENGINE=InnoDB;




-- TABELA ITENS DO PEDIDO


CREATE TABLE itens_pedido (
    id_item INT AUTO_INCREMENT PRIMARY KEY,
    id_pedido INT NOT NULL,
    id_produto INT NOT NULL,
    quantidade INT NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL,


    CONSTRAINT chk_quantidade CHECK (quantidade > 0),
    CONSTRAINT chk_preco_item CHECK (preco_unitario >= 0),
    UNIQUE (id_pedido, id_produto),


    FOREIGN KEY (id_pedido)
    REFERENCES pedidos(id_pedido)
    ON DELETE CASCADE,


    FOREIGN KEY (id_produto)
    REFERENCES produtos(id_produto)
) ENGINE=InnoDB;




-- TABELA PAGAMENTOS


CREATE TABLE pagamentos (
    id_pagamento INT AUTO_INCREMENT PRIMARY KEY,
    id_pedido INT NOT NULL,
    forma_pagamento VARCHAR(30) NOT NULL,
    valor DECIMAL(10,2) NOT NULL,
    status_pagamento VARCHAR(20) DEFAULT 'PENDENTE',
    data_pagamento DATETIME DEFAULT CURRENT_TIMESTAMP,


    CONSTRAINT chk_valor_pagamento CHECK (valor >= 0),


    FOREIGN KEY (id_pedido)
    REFERENCES pedidos(id_pedido)
) ENGINE=InnoDB;




-- TABELA HISTORICO DE STATUS


CREATE TABLE historico_status (
    id_historico INT AUTO_INCREMENT PRIMARY KEY,
    id_pedido INT NOT NULL,
    status_anterior VARCHAR(25),
    status_novo VARCHAR(25) NOT NULL,
    data_alteracao DATETIME DEFAULT CURRENT_TIMESTAMP,
    observacao VARCHAR(255),


    FOREIGN KEY (id_pedido)
    REFERENCES pedidos(id_pedido)
    ON DELETE CASCADE
) ENGINE=InnoDB;




-- =====================================================
-- 3. INSERIR DADOS FICTICIOS
-- =====================================================


INSERT INTO clientes
(nome, telefone, email, endereco)
VALUES
('Ana Souza','18990001001','ana@email.com','Rua das Flores, 100'),
('Bruno Lima','18990001002','bruno@email.com','Av. Brasil, 250'),
('Carla Mendes','18990001003','carla@email.com','Rua Central, 45'),
('Diego Alves','18990001004','diego@email.com','Rua Um, 80'),
('Eva Santos','18990001005','eva@email.com','Rua Dois, 90');




INSERT INTO restaurantes
(nome, categoria, telefone, endereco)
VALUES
('Burguer House','Hamburgueria','1833001001','Av. Principal, 500'),
('Pizzaria Bella','Pizzaria','1833001002','Rua Italia, 80'),
('Sushi Express','Japonesa','1833001003','Rua Oriente, 25'),
('Doce Encanto','Sobremesas','1833001004','Rua Acucar, 10'),
('Restaurante Novo','Brasileira','1833001005','Av. Norte, 700');




INSERT INTO entregadores
(nome, telefone, veiculo)
VALUES
('Marcos Oliveira','18991110001','Moto'),
('Julia Costa','18991110002','Bicicleta'),
('Pedro Rocha','18991110003','Moto');




INSERT INTO produtos
(id_restaurante, nome, descricao, preco)
VALUES
(1,'X-Burger','Hamburguer artesanal',25.90),
(1,'Batata frita','Porcao media',12.00),
(1,'Refrigerante','Lata 350 ml',6.00),
(2,'Pizza Mussarela','Pizza grande',48.00),
(2,'Pizza Calabresa','Pizza grande',52.00),
(3,'Combo Sushi','Combo com 20 pecas',45.00),
(3,'Temaki','Temaki de salmao',28.00),
(4,'Brownie','Brownie de chocolate',10.00);




INSERT INTO pedidos
(id_cliente, id_restaurante, id_entregador, status, taxa_entrega)
VALUES
(1,1,1,'PENDENTE',5.00),
(2,2,2,'PENDENTE',7.00),
(3,3,1,'PENDENTE',6.00),
(1,2,NULL,'PENDENTE',7.00),
(4,1,3,'PENDENTE',5.00);




INSERT INTO itens_pedido
(id_pedido, id_produto, quantidade, preco_unitario)
VALUES
(1,1,2,25.90),
(1,2,1,12.00),
(1,3,2,6.00),
(2,4,1,48.00),
(2,5,1,52.00),
(3,6,2,45.00),
(3,7,1,28.00),
(4,4,1,48.00),
(5,1,1,25.90);




INSERT INTO pagamentos
(id_pedido, forma_pagamento, valor, status_pagamento)
VALUES
(1,'PIX',80.80,'PENDENTE'),
(2,'CARTAO',107.00,'PENDENTE'),
(3,'DINHEIRO',124.00,'PENDENTE');




-- =====================================================
-- 4. UPDATE, DELETE E SELECT
-- =====================================================


-- CONSULTAR CLIENTES


SELECT * FROM clientes;


-- ATUALIZAR CLIENTE


UPDATE clientes
SET endereco = 'Rua Atualizada, 200'
WHERE id_cliente = 1;


-- INSERIR CLIENTE DE TESTE


INSERT INTO clientes
(nome, telefone, email, endereco)
VALUES
('Fabio Teste','18990001999','fabio@email.com','Rua Teste, 99');


-- EXCLUIR CLIENTE DE TESTE


DELETE FROM clientes
WHERE telefone = '18990001999';




-- =====================================================
-- 5. INNER JOIN 1
-- PEDIDOS POR CLIENTE E RESTAURANTE
-- =====================================================


SELECT
    p.id_pedido,
    c.nome AS cliente,
    r.nome AS restaurante,
    p.data_pedido,
    COUNT(ip.id_item) AS tipos_de_itens,
    SUM(ip.quantidade) AS unidades,
    SUM(ip.quantidade * ip.preco_unitario) AS subtotal


FROM pedidos p


INNER JOIN clientes c
ON c.id_cliente = p.id_cliente


INNER JOIN restaurantes r
ON r.id_restaurante = p.id_restaurante


INNER JOIN itens_pedido ip
ON ip.id_pedido = p.id_pedido


GROUP BY
    p.id_pedido,
    c.nome,
    r.nome,
    p.data_pedido


ORDER BY p.id_pedido;




-- =====================================================
-- 6. INNER JOIN 2
-- FATURAMENTO POR RESTAURANTE
-- =====================================================


SELECT
    r.id_restaurante,
    r.nome AS restaurante,
    COUNT(DISTINCT p.id_pedido) AS total_pedidos,
    SUM(ip.quantidade) AS unidades_vendidas,
    SUM(ip.quantidade * ip.preco_unitario) AS faturamento,
    AVG(ip.preco_unitario) AS preco_medio


FROM restaurantes r


INNER JOIN pedidos p
ON p.id_restaurante = r.id_restaurante


INNER JOIN itens_pedido ip
ON ip.id_pedido = p.id_pedido


GROUP BY
    r.id_restaurante,
    r.nome


ORDER BY faturamento DESC;




-- =====================================================
-- 7. LEFT JOIN
-- RESTAURANTES COM E SEM PEDIDOS
-- =====================================================


SELECT
    r.id_restaurante,
    r.nome AS restaurante,
    COUNT(DISTINCT p.id_pedido) AS quantidade_pedidos


FROM restaurantes r


LEFT JOIN pedidos p
ON p.id_restaurante = r.id_restaurante


GROUP BY
    r.id_restaurante,
    r.nome;




-- =====================================================
-- 8. VIEW
-- =====================================================


CREATE VIEW vw_resumo_pedidos AS


SELECT
    p.id_pedido,
    c.nome AS cliente,
    r.nome AS restaurante,
    p.data_pedido,
    p.status,
    COUNT(ip.id_item) AS tipos_de_itens,
    COALESCE(SUM(ip.quantidade),0) AS unidades,
    COALESCE(SUM(ip.quantidade * ip.preco_unitario),0) AS subtotal,
    p.taxa_entrega,
    p.valor_total


FROM pedidos p


INNER JOIN clientes c
ON c.id_cliente = p.id_cliente


INNER JOIN restaurantes r
ON r.id_restaurante = p.id_restaurante


LEFT JOIN itens_pedido ip
ON ip.id_pedido = p.id_pedido


GROUP BY
    p.id_pedido,
    c.nome,
    r.nome,
    p.data_pedido,
    p.status,
    p.taxa_entrega,
    p.valor_total;




-- TESTAR VIEW


SELECT * FROM vw_resumo_pedidos;




-- =====================================================
-- 9. FUNCTION
-- CALCULAR VALOR TOTAL DO PEDIDO
-- =====================================================


DELIMITER //


CREATE FUNCTION fn_calcular_total_pedido(p_id_pedido INT)
RETURNS DECIMAL(10,2)
READS SQL DATA
BEGIN


    DECLARE v_total DECIMAL(10,2) DEFAULT 0.00;


    SELECT
        COALESCE(SUM(ip.quantidade * ip.preco_unitario),0)
        + p.taxa_entrega


    INTO v_total


    FROM pedidos p


    LEFT JOIN itens_pedido ip
    ON ip.id_pedido = p.id_pedido


    WHERE p.id_pedido = p_id_pedido


    GROUP BY p.id_pedido, p.taxa_entrega;


    RETURN COALESCE(v_total,0.00);


END//


DELIMITER ;




-- TESTAR FUNCTION


SELECT fn_calcular_total_pedido(1);




-- =====================================================
-- 10. TRIGGER
-- REGISTRAR ALTERACOES DE STATUS AUTOMATICAMENTE
-- =====================================================


DELIMITER //


CREATE TRIGGER trg_historico_status


AFTER UPDATE ON pedidos


FOR EACH ROW


BEGIN


    IF OLD.status <> NEW.status THEN


        INSERT INTO historico_status
        (
            id_pedido,
            status_anterior,
            status_novo,
            observacao
        )


        VALUES
        (
            NEW.id_pedido,
            OLD.status,
            NEW.status,
            'Alteracao automatica pelo trigger'
        );


    END IF;


END//


DELIMITER ;




-- SEGUNDO TRIGGER: VALIDAR RESTAURANTE DO PRODUTO


DELIMITER //


CREATE TRIGGER trg_validar_produto


BEFORE INSERT ON itens_pedido


FOR EACH ROW


BEGIN


    DECLARE v_restaurante_pedido INT;
    DECLARE v_restaurante_produto INT;


    SELECT id_restaurante
    INTO v_restaurante_pedido
    FROM pedidos
    WHERE id_pedido = NEW.id_pedido;


    SELECT id_restaurante
    INTO v_restaurante_produto
    FROM produtos
    WHERE id_produto = NEW.id_produto;


    IF v_restaurante_pedido <> v_restaurante_produto THEN


        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT =
        'Produto nao pertence ao restaurante do pedido';


    END IF;


END//


DELIMITER ;




-- =====================================================
-- 11. PROCEDURE
-- FINALIZAR PEDIDO
-- =====================================================


DELIMITER //


CREATE PROCEDURE sp_finalizar_pedido(


    IN p_id_pedido INT,


    INOUT p_taxa DECIMAL(10,2),


    OUT p_mensagem VARCHAR(255)


)


BEGIN


    DECLARE v_existe INT DEFAULT 0;
    DECLARE v_status VARCHAR(25);
    DECLARE v_total DECIMAL(10,2);
    DECLARE v_itens INT DEFAULT 0;


    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_mensagem = 'Erro: transacao cancelada.';
        RESIGNAL;
    END;


    SET p_mensagem = '';


    IF p_taxa IS NULL OR p_taxa < 0 THEN


        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Taxa de entrega invalida';


    END IF;


    START TRANSACTION;


    SELECT COUNT(*)
    INTO v_existe
    FROM pedidos
    WHERE id_pedido = p_id_pedido;


    IF v_existe = 0 THEN


        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Pedido nao encontrado';


    END IF;


    SELECT status
    INTO v_status
    FROM pedidos
    WHERE id_pedido = p_id_pedido
    FOR UPDATE;


    IF v_status <> 'PENDENTE' THEN


        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Pedido nao esta pendente';


    END IF;


    SELECT COUNT(*)
    INTO v_itens
    FROM itens_pedido
    WHERE id_pedido = p_id_pedido;


    IF v_itens = 0 THEN


        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Pedido sem produtos';


    END IF;


    UPDATE pedidos
    SET taxa_entrega = p_taxa
    WHERE id_pedido = p_id_pedido;


    SET v_total = fn_calcular_total_pedido(p_id_pedido);


    UPDATE pedidos
    SET valor_total = v_total,
        status = 'CONFIRMADO'
    WHERE id_pedido = p_id_pedido;


    IF EXISTS (
        SELECT 1
        FROM pagamentos
        WHERE id_pedido = p_id_pedido
        AND status_pagamento = 'PENDENTE'
    ) THEN


        UPDATE pagamentos
        SET valor = v_total,
            forma_pagamento = 'PIX'
        WHERE id_pedido = p_id_pedido
        AND status_pagamento = 'PENDENTE';


    ELSE


        INSERT INTO pagamentos
        (id_pedido, forma_pagamento, valor, status_pagamento)
        VALUES
        (p_id_pedido, 'PIX', v_total, 'PENDENTE');


    END IF;


    COMMIT;


    SET p_mensagem = 'Pedido finalizado com sucesso!';


END//


DELIMITER ;




-- =====================================================
-- 12. TESTAR PROCEDURE
-- =====================================================


SET @taxa = 5.00;
SET @mensagem = '';


CALL sp_finalizar_pedido(1, @taxa, @mensagem);


SELECT
    @taxa AS taxa_utilizada,
    @mensagem AS mensagem;




-- CONFERIR PEDIDO


SELECT * FROM pedidos
WHERE id_pedido = 1;


-- CONFERIR PAGAMENTO


SELECT * FROM pagamentos
WHERE id_pedido = 1;




-- =====================================================
-- 13. TESTAR TRIGGER
-- =====================================================


UPDATE pedidos


SET status = 'EM_PREPARACAO'


WHERE id_pedido = 1;




-- CONSULTAR HISTORICO


SELECT * FROM historico_status
WHERE id_pedido = 1;




-- =====================================================
-- 14. SEGURANCA - CRIAR USUARIOS
-- =====================================================


CREATE USER IF NOT EXISTS
'gerente_delivery'@'localhost'
IDENTIFIED BY 'Gerente@2026';




CREATE USER IF NOT EXISTS
'atendente_delivery'@'localhost'
IDENTIFIED BY 'Atendente@2026';




-- PERMISSOES DO GERENTE


GRANT SELECT ON delivery_connect.*
TO 'gerente_delivery'@'localhost';


GRANT UPDATE (status) ON delivery_connect.pedidos
TO 'gerente_delivery'@'localhost';


GRANT EXECUTE ON FUNCTION
delivery_connect.fn_calcular_total_pedido
TO 'gerente_delivery'@'localhost';




-- PERMISSOES DO ATENDENTE


GRANT SELECT ON delivery_connect.clientes
TO 'atendente_delivery'@'localhost';


GRANT SELECT ON delivery_connect.restaurantes
TO 'atendente_delivery'@'localhost';


GRANT SELECT ON delivery_connect.produtos
TO 'atendente_delivery'@'localhost';


GRANT SELECT ON delivery_connect.pedidos
TO 'atendente_delivery'@'localhost';


GRANT INSERT ON delivery_connect.pedidos
TO 'atendente_delivery'@'localhost';


GRANT INSERT ON delivery_connect.itens_pedido
TO 'atendente_delivery'@'localhost';


GRANT EXECUTE ON PROCEDURE
delivery_connect.sp_finalizar_pedido
TO 'atendente_delivery'@'localhost';


GRANT EXECUTE ON FUNCTION
delivery_connect.fn_calcular_total_pedido
TO 'atendente_delivery'@'localhost';




-- CONFERIR PERMISSOES


SHOW GRANTS FOR 'gerente_delivery'@'localhost';


SHOW GRANTS FOR 'atendente_delivery'@'localhost';




-- REVOKE: RETIRAR PERMISSAO


REVOKE INSERT ON delivery_connect.pedidos
FROM 'atendente_delivery'@'localhost';




-- =====================================================
-- 15. CONFERENCIA FINAL
-- =====================================================


USE delivery_connect;


SHOW TABLES;


SELECT COUNT(*) AS total_clientes FROM clientes;


SELECT COUNT(*) AS total_restaurantes FROM restaurantes;


SELECT COUNT(*) AS total_produtos FROM produtos;


SELECT COUNT(*) AS total_pedidos FROM pedidos;


SELECT COUNT(*) AS total_itens FROM itens_pedido;


SELECT COUNT(*) AS total_pagamentos FROM pagamentos;


SELECT COUNT(*) AS total_historico FROM historico_status;


-- FIM DO PROJETO DELIVERY CONNECT
