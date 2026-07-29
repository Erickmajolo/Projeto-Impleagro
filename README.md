# Projeto-Impleagro
Projeto agricola para catalogos inteligentes
-- ============================================================
-- Catálogo Interativo de Peças Agrícolas
-- Estrutura completa do banco de dados (MVP) + dados de teste
-- Execute este script inteiro no SQL Editor do Supabase
-- ============================================================

-- Apaga as tabelas caso já existam (útil se você já rodou uma versão
-- anterior deste script e quer recomeçar do zero)
drop table if exists marcacoes cascade;
drop table if exists pecas cascade;
drop table if exists catalogos cascade;
drop table if exists modelos cascade;
drop table if exists fabricantes cascade;

-- ============================================================
-- TABELAS
-- ============================================================

-- Fabricante do implemento agrícola (ex: "Vence Tudo")
create table fabricantes (
  id uuid primary key default gen_random_uuid(),
  nome text not null,
  criado_em timestamp default now()
);

-- Modelo do implemento, pertence a um fabricante (ex: "Panther SM")
create table modelos (
  id uuid primary key default gen_random_uuid(),
  fabricante_id uuid not null references fabricantes(id) on delete cascade,
  nome text not null,
  criado_em timestamp default now()
);

-- Catálogo de peças (o PDF), pertence a um modelo
create table catalogos (
  id uuid primary key default gen_random_uuid(),
  modelo_id uuid not null references modelos(id) on delete cascade,
  nome text not null,
  arquivo_pdf text,          -- caminho do PDF no Storage (etapa futura)
  criado_em timestamp default now()
);

-- Peça cadastrada dentro de um catálogo
create table pecas (
  id uuid primary key default gen_random_uuid(),
  catalogo_id uuid not null references catalogos(id) on delete cascade,
  pagina integer,
  numero_item text,
  codigo text,
  descricao text,
  quantidade integer,
  observacoes text,
  criado_em timestamp default now()
);

-- Marcação visual da peça sobre o desenho (etapa futura de marcação)
create table marcacoes (
  id uuid primary key default gen_random_uuid(),
  peca_id uuid not null references pecas(id) on delete cascade,
  pagina integer,
  coordenada_x numeric,
  coordenada_y numeric,
  largura numeric,
  altura numeric,
  criado_em timestamp default now()
);

-- ============================================================
-- SEGURANÇA TEMPORÁRIA (apenas para fase de testes)
-- Antes de usar com dados reais, isso deve virar regras de
-- acesso por usuário autenticado.
-- ============================================================

alter table fabricantes enable row level security;
alter table modelos enable row level security;
alter table catalogos enable row level security;
alter table pecas enable row level security;
alter table marcacoes enable row level security;

create policy "acesso temporario teste" on fabricantes for all using (true) with check (true);
create policy "acesso temporario teste" on modelos for all using (true) with check (true);
create policy "acesso temporario teste" on catalogos for all using (true) with check (true);
create policy "acesso temporario teste" on pecas for all using (true) with check (true);
create policy "acesso temporario teste" on marcacoes for all using (true) with check (true);

-- ============================================================
-- DADOS DE TESTE
-- ============================================================

-- Fabricante: Vence Tudo
insert into fabricantes (id, nome)
values ('11111111-1111-1111-1111-111111111111', 'Vence Tudo');

-- Modelo: Panther SM (pertence ao fabricante acima)
insert into modelos (id, fabricante_id, nome)
values ('22222222-2222-2222-2222-222222222222',
        '11111111-1111-1111-1111-111111111111',
        'Panther SM');

-- Catálogo: Catálogo de peças Panther SM (pertence ao modelo acima)
insert into catalogos (id, modelo_id, nome)
values ('33333333-3333-3333-3333-333333333333',
        '22222222-2222-2222-2222-222222222222',
        'Catálogo de peças Panther SM');

-- Peça exemplo: Tubo injetor (pertence ao catálogo acima)
insert into pecas (catalogo_id, pagina, numero_item, codigo, descricao, quantidade, observacoes)
values ('33333333-3333-3333-3333-333333333333',
        47, '12', 'XXXXX', 'Tubo injetor', 1, null);

-- ============================================================
-- CONFERÊNCIA: consulta para ver tudo junto, como no exemplo do briefing
-- ============================================================
select
  f.nome as fabricante,
  m.nome as modelo,
  c.nome as catalogo,
  p.pagina,
  p.numero_item as item,
  p.codigo,
  p.descricao,
  p.quantidade
from pecas p
join catalogos c on c.id = p.catalogo_id
join modelos m on m.id = c.modelo_id
join fabricantes f on f.id = m.fabricante_id;
