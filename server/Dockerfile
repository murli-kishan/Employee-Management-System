FROM node:20-alpine

WORKDIR /app

ENV NODE_ENV=production

COPY package*.json ./

RUN npm ci --omit=dev

COPY index.js ./
COPY db/ ./db/
COPY models/ ./models/
COPY routes/ ./routes/
COPY utils/ ./utils/

EXPOSE 8000

CMD ["node", "index.js"]
