# ChatIA - Assistente Técnico com IA


MVP minimalista de um assistente de suporte técnico que conecta ao GEMINI para gerar respostas com base no que foi predefinido e guarda o histórico em MariaDB.
[ChatIA](https://chatia.labs-code.com/chat)


## Tecnologias
- FastAPI
- SQLAlchemy
- MariaDB
- GEMINI
- Docker

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
