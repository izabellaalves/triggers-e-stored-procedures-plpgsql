# PARCIAL SOBREPOSIÇÃO MAPEADO EM RELAÇÕES DIFERENTES

Inicialmente, cria-se as tabelas que servirão de base pros triggers e procedures.

```sql
-- Criando a tabela CEG
CREATE TABLE CEG (
    Ch SERIAL INT PRIMARY KEY,
    AG VARCHAR(255) 
);

-- Criando a tabela CEE1
CREATE TABLE CEE1 (
    Ch INT PRIMARY KEY,
    Ae1 VARCHAR(255) 

    FOREING KEY (Ch) REFERENCES CEG(Ch)
); 
```

Neste caso, não há nada que o usuário possa fazer que gere inconsistência, pois não há como inserir em CEE1 sem antes inserir em CEG. Portanto, não há necessidade de criar procedures para inserção.

O único caso é se houver alguma regra de negócio, por exemplo, suponha que exista mais 2 tabelas, CEE2 e CEE3, e que está tudo bem se o usuário estiver em CEE1 e em CEE2, mas não pode estar em CEE1 e em CEE3. Neste caso, é necessário criar uma procedure para inserção em CEE1 e CEE3, que verifique se o usuário já está em uma das tabelas antes de inserir.

Faremos aqui um exemplo de procedure para inserção em CEE1, que verifica se ele já existe em CEE3.

```sql

CREATE OR REPLACE FUNCTION insere_CEE1() RETURNS TRIGGER AS $$ 

BEGIN 

    PERFORM * FROM CEE3 WHERE Ch = NEW.Ch;
    IF FOUND THEN
        RAISE EXCEPTION 'Usuário já está em CEE3';
    END IF;

    RETURN NEW;

END

$$ LANGUAGE plpgsql;

CREATE TRIGGER insere_CEE1 BEFORE INSERT ON CEE1 FOR EACH ROW EXECUTE PROCEDURE insere_CEE1();

```

