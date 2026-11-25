# Тестова матрица - GameStudio NEO

## Изискване 1: CI/CD към Game Server

- `hq -> Game Server:22` - МИНАВА
- `hq -> Game Server:7777` - МИНАВА

## Изискване 2: QA към Game Server

- QA и Game Server са в една мрежа, не минава през gw

## Изискване 3: Dev към Jenkins

- `vtarnovo -> Jenkins:8080` - МИНАВА (само GET, ниво 7)

## Изискване 4: Game Server към Dev мрежи

- `gabrovo -> Jenkins:8080` - НЕ МИНАВА
- `gabrovo -> ssh-dev:22` - НЕ МИНАВА

## Изискване 5: DNS към 8.8.8.8

- Не може да се наложи с nftables в тази симулация
- Записано като ограничение в матрицата

## Журнал

- `[GAMESTUDIO-DROP] IN=v-gabrovo OUT=v-hq ... DPT=8080`
- `[GAMESTUDIO-DROP] IN=v-gabrovo OUT=v-vtarnovo ... DPT=22`
