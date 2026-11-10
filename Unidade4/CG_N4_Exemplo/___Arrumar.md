      if (estadoTeclado.IsKeyPressed(Keys.P) && objetoSelecionado != null)
        Console.WriteLine(objetoSelecionado.ToString());
Quando o Cubo está selecioando não imprime os dados, só: gcgcg.Cubo

Parece que o Depth Test não está funcionando
O cubo pequeno deveria ficar na frente do cubo grande, mas sempre aparece só o cubo grande.
Testar usando as teclas O e L para "levar" o cubo pequeno para frente e para trás.

A tecla Home (ou End) amplia o cubo grande, mas não está ampliando o cubo peqeuno
Testar ampliar o cubo pequeno, e depois rotacionar usando 3/4 para ver se gira em torno de si mesmo.

