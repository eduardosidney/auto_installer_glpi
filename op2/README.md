Próximas atividades a serem inseridas para melhorar a funcionalidade do script



Adicionar o padrão "erre e saia" (fail fast), para evitar que o script continue a executar caso algum comando tenha falhado. "set -e" no início do script para habilitar;

Guardar as mensagem de saída em variaveis para que possa usar novamente;

Criar funções para encapsular lógicas separadas;.

Otimize as saídas

Remover a saída na tela, direcionando as saídas para /dev/null (2>/dev/null)
