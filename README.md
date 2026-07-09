git clone --recurse-submodules

## Mettre à jour le projet
git pull
git submodule update --init --recursive
git submodule foreach 'git checkout develop && git pull origin develop'

## Mettre à jour les DTO en proto
cd modules/proto
npm install
npm run generate
