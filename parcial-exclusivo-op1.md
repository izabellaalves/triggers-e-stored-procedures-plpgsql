# PARCIAL EXCLUSIVO MAPEADO EM RELAÇÕES DIFERENTES

Inicialmente, cria-se as tabelas que servirão de base pros triggers e procedures.

```sql
-- Criando a tabela CEG
CREATE TABLE CEG (
    Ch SERIAL PRIMARY KEY,
    AtC CHAR(1),
    AG VARCHAR(20)
);

-- Criando a tabela CEE1
CREATE TABLE CEE1 (
    Ch INTEGER PRIMARY KEY,
    Aei VARCHAR(20),
    -- Adicionando chave estrangeira referenciando a tabela CEG
    FOREIGN KEY (Ch) REFERENCES CEG(Ch)
);

```
Alguns problemas podem acontecer neste caso:

* O usuário pode inserir em CEG com o AtC errado.
* O usuário pode inserir em várias tabelas específicas com o mesmo ID.
* O usuário pode alterar o AtC na tabela CEG para um valor errado.

Para tratar este caso, revoga-se o direito do usuário de inserção nas tabelas, e cria-se um procedimento que faz a inserção em cada uma das tabelas.

- Insere em CEG
```sql

CREATE OR REPLACE PROCEDURE insere_CEG(
    AG VARCHAR(20)
) AS $$

BEGIN

    INSERT INTO CEG(AtC, AG) VALUES (NULL, AG);

COMMIT;

END

$$ LANGUAGE plpgsql;

```

- Insere em CEE1:

```sql
CREATE OR REPLACE PROCEDURE insere_CEE1(
    AG VARCHAR(20),
    Ae1 VARCHAR(20)
) AS $$

DECLARE 
    Ch_CEG INTEGER;

BEGIN
    INSERT INTO CEG(Atc, AG) VALUES (1, AG) RETURNING Ch INTO Ch_CEG;
    INSERT INTO CEE1(Ch, Ae1) VALUES (Ch_CEG, Ae1);

    COMMIT;
END

$$ LANGUAGE plpgsql;
```

Além disso, cria-se um trigger que revoga o direito de alteração do AtC e da chave na tabela CEG.

```sql

CREATE OR REPLACE FUNCTION revoga_AtC_Ch() RETURNS TRIGGER AS $$
BEGIN
    IF (OLD.AtC <> NEW.AtC && OLD.Ch <> NEW.Ch) THEN
        RAISE EXCEPTION 'Não é possível alterar o AtC e nem a chave primária da tabela CEG!';
    END IF;
    RETURN NEW;
END

$$ LANGUAGE plpgsql;

CREATE TRIGGER revoga_AtC_Ch
BEFORE UPDATE ON CEG
FOR EACH ROW
EXECUTE PROCEDURE revoga_AtC_Ch();
```

E cria-se um trigger que revoga o direito de modificar o ID da tabela CEE1.

```sql

CREATE OR REPLACE FUNCTION revoga_CH_CEE1() RETURNS TRIGGER AS $$

BEGIN

IF (OLD.Ch <> NEW.Ch) THEN
    RAISE EXCEPTION 'Não é possível alterar a chave!';
END IF;

RETURN NEW;

END

$$ LANGUAGE plpgsql;

CREATE TRIGGER revoga_CH_CEE1 
BEFORE UPDATE ON CEE1
FOR EACH ROW
EXECUTE PROCEDURE revoga_CH_CEE1();
```