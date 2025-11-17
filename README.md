# Assistente Técnico com IA - (Projeto de portfólio)


MVP minimalista de um assistente de suporte técnico que conecta ao GEMINI para gerar respostas com base no que foi predefinido e guarda o histórico em MariaDB.
[Clique aqui para acessar o ChatIA](https://chatia.labs-code.com/chat)


## Tecnologias
![Python](https://img.shields.io/badge/Python-%23254f73.svg?style=for-the-badge&logo=python&logoColor=yellow)
![FastAPI](https://img.shields.io/badge/fastapi-%23009485.svg?style=for-the-badge&logo=fastapi&logoColor=white)
![HTML5](https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/css3-%231572B6.svg?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E)
![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)
![MariaDB](https://img.shields.io/badge/mariadb-%234e629A.svg?style=for-the-badge&logo=mariadb&logoColor=%23c0765a) 
![GEMINI](https://img.shields.io/badge/gemini-%23131313.svg?style=for-the-badge&logo=gemini&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%2300153c.svg?style=for-the-badge&logo=docker&logoColor=white)

## Tree do projeto
```cmd

../chatia/
|-- Dockerfile
|-- README.md
|-- app
|   |-- crud.py
|   |-- database.py
|   |-- main.py
|   |-- models.py
|   |-- schemas.py
|   |-- static
|   |   `-- style.css
|   `-- templates
|       `-- index.html
|-- docker-compose.yml
`-- requirements.txt
```

## Trechos do código
### Rotas e configuração da persona da IA
```python
@app.get("/", response_class=RedirectResponse)
async def root():
    """Redireciona a raiz para a página de chat."""
    return RedirectResponse(url="/chat", status_code=302)

@app.get("/chat", response_class=HTMLResponse)
async def chat_page(request: Request, db: Session = Depends(get_db)):
    """Exibe a página principal do chat e carrega o histórico."""
    # Note: O histórico está em ordem decrescente (mais recente primeiro)
    history = crud.get_last_conversations(db, limit=50) 
    return templates.TemplateResponse("index.html", {"request": request, "history": history})


@app.post("/ask", response_class=HTMLResponse)
async def ask(request: Request, user_input: str = Form(...), db: Session = Depends(get_db)):
    """Recebe a pergunta do usuário, gera a resposta da IA e salva no banco de dados."""
    
    # Mensagem Padrão (Mock)
    ai_response = "(Resposta mock) Sugestão: verifique os logs do servidor, reinicie o serviço e cheque permissões."
    
    # Tenta usar o modelo Gemini
    if client:
        try:
            # Define a persona e a entrada do usuário
            prompt = f"Você é um assistente técnico experiente em infraestrutura de TI. Responda apenas à pergunta do usuário. \nPergunta do Usuário: {user_input}"
            
            # Chama o método de inferência do Gemini
            response = client.models.generate_content(
                model=GEMINI_MODEL,
                contents=prompt,
                config={
                    "max_output_tokens": 1024 # Evita respostas muito longas
                }
            )
            
            # Extrai o texto gerado
            ai_response = response.text
```

### Model
```python
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.sql import func
from .database import Base

class Conversation(Base):
    __tablename__ = "conversations"

    id = Column(Integer, primary_key=True, index=True)
    user_input = Column(String(1024))
    ai_response = Column(String(4096))
    timestamp = Column(DateTime, default=func.now())

    def __repr__(self):
        return f"<Conversation(id={self.id}, user_input='{self.user_input[:20]}...')>"
```
