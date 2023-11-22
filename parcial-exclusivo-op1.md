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

Para tratar este caso, revoga-se o direito do usuário de inserção nas tabelas, e cria-se um procedimento que faz a inserção nas duas tabelas.

```sql
CREATE OR REPLACE PROCEDURE insere_CEEi(
    AtC CHAR(1),
    AG VARCHAR(20),
    Ae1 VARCHAR(20)
) AS $$

DECLARE 
    Ch_CEG INTEGER;
BEGIN
    IF (AtC = '0') THEN
        INSERT INTO CEG(Atc, AG) VALUES (0, AG);
    END IF;

    IF (AtC = '1') THEN
        INSERT INTO CEG(Atc, AG) VALUES (1, AG) RETURNING Ch INTO Ch_CEG;
        INSERT INTO CEE1(Ch, Ae1) VALUES (Ch_CEG, Ae1);
    END IF;

    COMMIT;
END

$$ LANGUAGE plpgsql;
```

Além disso, cria-se um trigger que revoga o direito de alteração do AtC na tabela CEG.

```sql

CREATE OR REPLACE FUNCTION revoga_AtC() RETURNS TRIGGER AS $$
BEGIN
    IF (OLD.AtC <> NEW.AtC) THEN
        RAISE EXCEPTION 'Não é possível alterar o AtC';
    END IF;
    RETURN NEW;
END

$$ LANGUAGE plpgsql;

CREATE TRIGGER revoga_AtC
BEFORE UPDATE ON CEG
FOR EACH ROW
EXECUTE PROCEDURE revoga_AtC();

```
