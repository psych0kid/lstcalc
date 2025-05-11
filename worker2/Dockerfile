FROM golang:1.23-rc-alpine
WORKDIR /worker2
COPY . .
RUN go build -o main .
CMD ["./main"]
