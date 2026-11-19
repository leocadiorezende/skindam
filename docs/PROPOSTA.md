# Proposta: automação de DMs no Instagram

Uma ferramenta própria, no estilo ManyChat, para enviar DM automática quando alguém comenta uma palavra-chave num post.

## Como funciona para o seguidor

1. A pessoa comenta **"quero"** num post.
2. O perfil responde o comentário publicamente: *"Te mandei no direct! 📩"*.
3. A pessoa recebe uma DM com um botão: **"Quero receber o link"**.
4. Quando ela toca no botão, o sistema verifica se ela segue o perfil (campo `is_user_follow_business` da [API de perfil do Instagram](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/messaging-api/user-profile)).
   - Se não segue: recebe *"Para liberar, siga a Skin Dam e toque em 'Já segui'"*, e o sistema verifica de novo.
   - Se segue: chega a DM com o link ou o arquivo daquele post.

A exigência de seguir é opcional e se liga ou desliga em cada post. O painel também conta quantas pessoas seguiram o perfil para liberar o conteúdo.

O botão do passo 3 serve para garantir a entrega. A Meta coloca a primeira DM de quem **não segue** o perfil na pasta "Solicitações" ([documentação da Meta](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/messaging-api/private-replies)). Quando a pessoa toca no botão, a conversa é aberta de verdade e o link chega na caixa principal. O ManyChat usa a mesma técnica.

## O que ele controla no painel

O painel é uma página web com senha, que funciona no celular e no computador.

- **Posts:** mostra os posts recentes, com miniatura, puxados direto do Instagram.
- **Para cada post:**
  - palavra-chave (uma ou mais, por exemplo "quero", "link", "eu");
  - resposta pública no comentário (pode ter algumas versões, que se alternam);
  - mensagem com o botão de confirmação;
  - mensagem final com o link ou arquivo;
  - botão para ligar e desligar a automação.
- **Arquivos:** ele envia um PDF ou e-book pelo painel e o sistema gera o link automaticamente.
- **Proteção:** a mesma pessoa não recebe a mesma DM duas vezes no mesmo post.

## Números (por post e por período)

| Métrica | O que mostra |
|---|---|
| Comentários com a palavra-chave | Quantas pessoas pediram |
| DMs enviadas | Quantas receberam a primeira mensagem |
| **Cliques no botão "Quero receber"** | Quantas confirmaram e receberam o link ou arquivo |
| Cliques no link / downloads | Quantas abriram de fato o conteúdo |
| Taxa de conversão | % de quem comentou e chegou até o link |

Os cliques no link são contados porque cada link passa por um endereço próprio do sistema antes de redirecionar para o destino. O seguidor não percebe a diferença.

## Quanto custa por mês

### Hoje (100 a 150 comentários por dia)

| Item | Serviço | Custo mensal |
|---|---|---|
| Servidor + painel | Cloudflare Workers (plano grátis: 100 mil requisições/dia) | **R$ 0** |
| Banco de dados | Cloudflare D1 (plano grátis) | **R$ 0** |
| Armazenamento de arquivos (PDFs etc.) | Cloudflare R2 (plano grátis: 10 GB) | **R$ 0** |
| API do Instagram | Meta | **R$ 0** |
| Endereço do painel | `nome.workers.dev` | **R$ 0** |
| Domínio próprio (opcional) | `.com.br` no Registro.br, R$ 40/ano | R$ 3,33 |
| **Total** | | **R$ 0 a R$ 3,33/mês** |

Cada comentário gera cerca de 5 requisições no servidor. Com 150 comentários por dia, o uso fica perto de 750 requisições por dia, menos de 1% do limite grátis.

### Se ele crescer

| Comentários por dia | Custo mensal estimado |
|---|---|
| até ~15.000 | R$ 0 |
| acima disso | Cloudflare Workers Paid: US$ 5/mês (~R$ 29) |

### Comparação com o ManyChat

O ManyChat cobra pelo número de pessoas que interagem por mês ("contatos ativos"). Com 100 a 150 comentários por dia, isso dá cerca de 3.000 a 4.500 contatos por mês.

| Volume mensal | Plano ManyChat | Custo em US$ | Custo em R$* |
|---|---|---|---|
| 3.000 contatos | Pro (2.500 incluídos + US$ 0,05 por extra) | ~US$ 64 | ~R$ 375 |
| 4.500 contatos | Business (até 7.500) | US$ 99 | ~R$ 580 |
| 10.000 contatos | Business + extras | ~US$ 160 | ~R$ 940 |

\* Dólar a R$ 5,86 (fechamento de 18/11/2026), sem IOF do cartão internacional. Preços do ManyChat em [Hyperleap](https://hyperleap.ai/comparison/manychat-pricing).

**Economia estimada: R$ 4.500 a R$ 7.000 por ano**, considerando o volume atual.

## Manutenção

O projeto foi pensado para dar o mínimo de trabalho:

- **Sem servidor para cuidar:** o Cloudflare mantém a infraestrutura, sem atualizações de sistema nem máquina para reiniciar.
- **Deploy automático:** qualquer alteração enviada para o GitHub entra no ar sozinha.
- **Token renovado automaticamente:** o acesso ao Instagram expira a cada 60 dias, e o sistema renova antes disso.
- **Aviso de problema:** se a conexão com o Instagram cair, o painel mostra um alerta.
- **Manutenção esperada:** perto de zero no dia a dia. A Meta atualiza a versão da API mais ou menos a cada 2 anos, o que exige um ajuste pequeno.

## O que é preciso para começar

- [ ] Instagram em modo **profissional** (Criador ou Empresa). É grátis e se troca nas configurações.
- [ ] Conta de desenvolvedor na Meta. É grátis, **não precisa de CNPJ** e usa um Facebook pessoal.
- [ ] Conta no Cloudflare. É grátis e não pede cartão.
- [ ] Repositório no GitHub (já existe).

## Limitações do Instagram (valem para qualquer ferramenta, inclusive o ManyChat)

- **Janela de 7 dias:** a DM automática só pode ser enviada até 7 dias depois do comentário.
- **Janela de 24 horas:** depois que a pessoa interage, dá para continuar a conversa por 24 horas.
- **Sem disparo em massa:** não é possível mandar mensagem para quem nunca interagiu.
- **TikTok:** existe uma API de mensagens, mas ela está em beta e depende de aprovação do TikTok. Fica para uma segunda fase, se a aprovação sair.
