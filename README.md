# Desafio-Analise-Stride-Usando-IA-DIO
Projeto para o desafio da DIO sobre análise de arquitetura com Python, FastAPI e Azure OpenAI.

Projeto: Análise de Ameaças STRIDE com Python e IA Generativa

Desafio da Formação Python AI Backend Developer da DIO

--> Qual o objetivo desse projeto?
Esse repositório é o resultado de um desafio da plataforma DIO, onde a proposta era desenvolver e conectar algumas tecnologias que estão em alta no mercado: Python, FastAPI e o poder da IA Generativa da Azure.

A ideia central foi construir uma API que faz algo que, até pouco tempo, parecia ficção científica: ela recebe uma imagem de um diagrama de arquitetura de software, "conversa" com uma IA que consegue enxergar e interpretar essa imagem, e devolve uma análise de riscos de segurança baseada na metodologia STRIDE.

Pra mim, foi a oportunidade perfeita para ir além da teoria e ver, na prática, como tudo isso funciona para criar uma solução com valor real.

🚀 As Ferramentas e o Aprendizado:
Durante o projeto, trabalhei com várias tecnologias. Mais do que só usar, tentei entender o "porquê" de cada uma. Aqui estão minhas impressões:

Python e FastAPI: Sinceramente, o FastAPI foi uma surpresa. Eu já tinha ouvido falar, mas usar no dia a dia é outra coisa. A velocidade de desenvolvimento que ele proporciona e, principalmente, a documentação interativa que ele gera automaticamente (o Swagger) é algo que economiza muito tempo. Facilitou demais na hora de testar o endpoint.

Azure OpenAI (GPT-4 Vision): Aqui eu pude realmente entender sobre o poder da IA hoje. Uma coisa é usar o ChatGPT, outra é integrar a API dele num sistema meu. Ver o modelo receber uma imagem que eu enviei e descrever os componentes dela com precisão foi o momento mais marcante do projeto. Fica claro o potencial disso pra automatizar tarefas de análise que hoje são feitas de forma manual.

Engenharia de Prompt: Essa parte foi um aprendizado a parte. No começo, eu fazia algumas instruções simples e a IA me devolvia umas respostas bem genéricas. Eu percebi, então, que engenharia de prompt é quase como programar em linguagem natural. Tive que fazer e refazer o prompt várias vezes, dando um papel específico pra IA ("você é um analista de segurança sênior", ou "você é um especialista em segurança com mais de 10 anos de mercado"), estruturando como eu queria a resposta... Foi um processo de tentativa e erro que me ensinou a ser muito mais claro e direto nas minhas instruções.

Metodologia STRIDE: Eu já conhecia o conceito de modelagem de ameaças, mas o STRIDE me deu um framework prático. É quase um checklist mental pra olhar pra uma arquitetura e começar a perguntar: "Onde alguém poderia se passar por outro usuário? Onde os dados poderiam ser alterados? E se o serviço sair do ar?". Ter essa estrutura guiou toda a lógica do prompt e deu um propósito real para a análise da IA.

⚙️ Como a "Mágica" Acontece: A Arquitetura
Se fosse pra desenhar o fluxo "para uma criança entender", seria assim:

O Começo (Upload): Tudo começa quando alguém envia um arquivo de imagem (um .PNG ou .JPG de um diagrama) para a API que eu criei.

O Preparo (FastAPI): A API pega essa imagem, confere pra ver se é uma imagem mesmo e faz uma leve mudança: converte ela para um formato de texto (base64). Isso é necessário pra poder mandar a imagem pela internet dentro de um pacote de dados padrão (JSON).

A Pergunta (Azure OpenAI): A nossa aplicação, então, monta uma mensagem para a IA. Essa mensagem tem duas partes: o nosso prompt super detalhado (as instruções) e a imagem já "traduzida" para base64.

A Resposta Inteligente: A IA no Azure recebe tudo isso, analisa a imagem seguindo as regras que a gente mandou no prompt e elabora a análise de ameaças STRIDE.

De Volta pra Casa: A API recebe essa análise de volta (que é só um texto bem formatado), empacota num JSON e entrega como resposta para quem fez a requisição lá no começo. Simples e eficiente.

💻 Um Pedaço do Código pra Ilustrar
Não vou colocar o código todo aqui, mas acho que vale mostrar duas partes que são o coração de tudo isso:

1. Recebendo o arquivo com FastAPI
Python

# A gente define a "porta de entrada" da API aqui.
# É um endpoint POST que espera um arquivo.
@app.post("/analyze-architecture/")
async def analyze_architecture(file: UploadFile = File(...)):
    # Um check básico de segurança e sanidade
    if not file.content_type.startswith('image/'):
        raise HTTPException(status_code=400, detail="Opa, isso não parece uma imagem.")

    # A gente lê o arquivo de forma assíncrona pra não travar a aplicação
    image_contents = await file.read()
    
    # ... aqui a gente chama a lógica da IA ...
    
    return {"analysis": "A análise completa da IA viria aqui."}
O legal do FastAPI é que com esse pedaço de código, ele já executa muita coisa do que precisamos.

2. "Conversando" com a API Vision
Python

# Essa é a parte que monta o pacote pra enviar pra IA.
response = client.chat.completions.create(
    model=NOME_DO_MODELO_NO_AZURE,
    messages=[
        {
            "role": "user",
            # O conteúdo é uma lista: primeiro o texto (nosso prompt), depois a imagem.
            "content": [
                {"type": "text", "text": MEU_PROMPT_DETALHADO},
                {
                    "type": "image_url",
                    "image_url": { "url": f"data:image/jpeg;base64,{imagem_em_base64}" }
                }
            ]
        }
    ],
    max_tokens=3000
)

# A resposta que a gente quer está dentro do objeto de resposta
analysis_content = response.choices[0].message.content
Esse trecho foi o que exigiu mais pesquisa. Entender exatamente como formatar essa mensagem para o modelo de IA foi uma das chaves do projeto.

🧠 Os desafios e o que eu aprendi disso tudo
Chaves de API e Segurança: Confesso que a primeira dificuldade foi pensar em onde eu coloco essas chaves de API sem deixar elas expostas no código? Isso me forçou a aprender sobre variáveis de ambiente e a importância de nunca, jamais, em hipótese alguma subir senhas e chaves para um repositório público. É o óbvio, porém o óbvio precisa ser dito e precisa ser levado a sério.

A IA é excelente, uma ótima ferramenta, porém não faz milagre: O maior aprendizado técnico foi que a IA é uma ferramenta maravilhosa, entretanto ela precisa de todos os detalhes no prompt. O retorno que eu recebia era um reflexo direto da qualidade do meu prompt (garbage in, garbage out - ou, lixo dentro, lixo fora; isso significa que, se eu fornecer um prompt com informações erradas, a IA retornará um código incorreto e com erros). Foi um exercício prático de comunicação e especificação.

Conexão dos Pontos: O mais legal foi ver como as coisas se conectam. Uma API bem feita, um serviço de nuvem poderoso e um problema real de negócio (análise de segurança). Esse projeto me deu uma visão muito mais clara de como eu posso usar a programação para construir ferramentas que automatizam tarefas complexas e inteligentes para serem utilizadas em diversos projetos no mercado de tecnologia.

🏁 Meus Próximos Passos
Esse projeto foi um aprendizado excepcional, um verdadeiro projeto teórico-prático. Fico com a sensação de que, mais do que só entregar um desafio, eu realmente construí uma base sólida de conhecimento.

Consequentemente, já comecei a pensar em como melhorar:

- Criar uma interface visual simples: Um front-end básico onde qualquer um possa simplesmente arrastar e soltar a imagem para ver a análise;

- Testar com diagramas mais complexos: Usar diagramas de arquiteturas reais, com mais componentes, para testar toda a capacidade da IA.

- Adicionar um cache: Para não gastar dinheiro à toa processando a mesma imagem várias vezes.

- Entre outros projetos (por mais que a tecnologia já esteja absurdamente desenvolvida atualmente, com certeza ainda temos muito o que aprender e desenvolver!!!)

No mais, foi um projeto extremamente divertido e desafiador. Agradeço à DIO pela oportunidade de poder aprender e aplicar esse conhecimento de forma tão prática.
