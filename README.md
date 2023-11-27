## Biblioteca_Comercio

![image](https://github.com/soaresy/Biblioteca_Comercio/assets/144077766/e4c5a057-3909-4d1c-aa5e-e91ef796dad3)



## Codigo


DELIMITER //

CREATE PROCEDURE RegistrarEmprestimo(
    IN Livro_ID INT,
    IN Cliente_ID INT,
    IN Data_Emprestimo DATE,
    IN Data_Devolucao DATE
)
BEGIN
    DECLARE Livro_Disponivel INT;
    SELECT COUNT(*) INTO Livro_Disponivel FROM Livros WHERE ID = Livro_ID AND ID NOT IN (SELECT LivroID FROM Emprestimos WHERE Status = 'pendente');

    IF Livro_Disponivel > 0 THEN
        INSERT INTO Emprestimos (LivroID, ClienteID, DataEmprestimo, DataDevolucao, Status)
        VALUES (Livro_ID, Cliente_ID, Data_Emprestimo, Data_Devolucao, 'pendente');

        UPDATE Livros SET Disponivel = 0 WHERE ID = Livro_ID;
    ELSE
        SELECT 'Livro não disponível para empréstimo';
    END IF;
END //

CREATE PROCEDURE ListaLivrosPorCliente(
    IN Cliente_ID INT
)
BEGIN
    SELECT Livros.Titulo, Livros.ISBN, Livros.AnoPublicacao, Autores.Nome as NomeAutor
    FROM Livros
    INNER JOIN LivroAutor ON Livros.ID = LivroAutor.LivroID
    INNER JOIN Autores ON LivroAutor.AutorID = Autores.ID
    INNER JOIN Emprestimos ON Livros.ID = Emprestimos.LivroID
    WHERE Emprestimos.ClienteID = Cliente_ID;
END //

CREATE PROCEDURE CalcularMultaAtraso()
BEGIN
    UPDATE Emprestimos
    SET Multa = 5 * DATEDIFF(CURDATE(), DataDevolucao)
    WHERE DataDevolucao < CURDATE() AND Status = 'pendente';
END //

DELIMITER ;


CREATE VIEW LivrosDisponiveis AS
SELECT *
FROM Livros
WHERE ID NOT IN (SELECT LivroID FROM Emprestimos WHERE Status = 'pendente');

CREATE VIEW ListaEmprestimos AS
SELECT Livros.Titulo, Livros.ISBN, Emprestimos.DataEmprestimo, Emprestimos.DataDevolucao, Emprestimos.Status, Emprestimos.ClienteID
FROM Livros
INNER JOIN Emprestimos ON Livros.ID = Emprestimos.LivroID;
