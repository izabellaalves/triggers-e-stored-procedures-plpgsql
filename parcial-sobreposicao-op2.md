# PARCIAL SOBREPOSIÇÃO MAPEADO EM RELAÇÕES DIFERENTES 2

Inicialmente, cria-se as tabelas que servirão de base pros triggers e procedures.

```sql

CREATE TABLE CEG (
    Ch SERIAL INT PRIMARY KEY,
    AG VARCHAR(255) 
);

CREATE TABLE CEE1 (
    Ch INT PRIMARY KEY,
    Ae1 VARCHAR(255) 

    FOREING KEY (Ch) REFERENCES CEG(Ch)
); 

CREATE TABLE CEC (
    Ch INT,
    AtC CHAR(1)

    FOREING KEY (Ch) REFERENCES CEG(Ch)
    PRIMARY KEY (Ch, AtC);
); 

```

Neste caso, possuímos as seguintes possíveis inconsistências:

* O usuário pode inserir em CEC com o AtC errado.
* O usuário pode modificar a chave primária de CEE1.

Por isso, é necessário revogar o direito de inserção nas tabelas, e criar procedures para inserção. Cria-se, então, uma procedure para inserção em CEG, e uma procedure para inserção em CEE1.

- Insere em CEG:

```sql

CREATE OR REPLACE PROCEDURE insere_CEG(
    AG VARCHAR(255)
) AS $$

DECLARE
    Ch_CEG INTEGER;

BEGIN

    INSERT INTO CEG (AG) VALUES (AG) RETURNING Ch INTO Ch_CEG;

    INSERT INTO CEC (Ch, Atc) VALUES (Ch_CEG, NULL);

COMMIT;

END;

$$ LANGUAGE plpgsql;

```

- INSERE EM CEE1:

```sql
CREATE OR REPLACE PROCEDURE insere_CEE1(
    AG VARCHAR(20),
    Ae1 VARCHAR(20)
) AS $$

DECLARE 
    Ch_CEG INTEGER;

BEGIN

    INSERT INTO CEG(AG) VALUES (AG) RETURNING Ch INTO Ch_CEG;

    INSERT INTO CEE1(Ch, Ae1) VALUES (Ch_CEG, Ae1);

    INSERT INTO CEC(Ch, AtC) VALUES (Ch_CEG, 1);

COMMIT;

END;

$$ LANGUAGE plpgsql;
```

Além disso, o usuário pode tentar modificar a chave primária de CEE1, o que não é permitido. Para isso, cria-se um trigger que revoga o direito de alteração da chave primária de CEE1.

```sql

CREATE OR REPLACE FUNCTION revoga_Ch() RETURNS TRIGGER AS $$

BEGIN
    IF (OLD.Ch <> NEW.Ch) THEN
        RAISE EXCEPTION 'Não é possível alterar a chave primária da tabela CEE1!';
    END IF;
    RETURN NEW;
END

$$ LANGUAGE plpgsql;

CREATE TRIGGER revoga_Ch BEFORE UPDATE ON CEE1 FOR EACH ROW EXECUTE PROCEDURE revoga_Ch();
```

O usuário também pode tentar alterar a chave ou o AtC em CEC, o que não é permitido. Para isso, cria-se um trigger que revoga o direito de alteração da chave e do AtC de CEC.

```sql

