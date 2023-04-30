#include <Trade\Trade.mqh>
#include <Trade\PositionInfo.mqh>
#include <Trade\SymbolInfo.mqh>

input int MaxOrders = 5;
input double Distance = 100;
input double Lots = 0.01;

void OnTick()
{
    double current_price = SymbolInfoDouble(_Symbol, SYMBOL_BID);
    static double prev_price = 0;
    static int orders_count = 0;

    // Controllo se il prezzo si è invertito rispetto al prezzo di apertura delle posizioni
    if ((current_price > prev_price && orders_count > 0 && orders_count == MaxOrders) || (current_price < prev_price && orders_count > 0 && orders_count == MaxOrders))
    {
        int total = PositionsTotal();

        for (int i = total - 1; i >= 0; i--)
        {
            if (PositionSelectByTicket(PositionGetTicket(i)))
            {
                if ((PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY && current_price < PositionGetDouble(POSITION_PRICE_OPEN)) || (PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL && current_price > PositionGetDouble(POSITION_PRICE_OPEN)))
                {
                    MqlTradeRequest close_request = { TRADE_ACTION_DEAL };
                    MqlTradeResult close_result = {0};

                    close_request.action = TRADE_ACTION_DEAL;
                    close_request.type = PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY ? ORDER_TYPE_SELL : ORDER_TYPE_BUY;
                    close_request.symbol = PositionGetString(POSITION_SYMBOL);
                    close_request.volume = PositionGetDouble(POSITION_VOLUME);
                    close_request.price = SymbolInfoDouble(close_request.symbol, PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY ? SYMBOL_BID : SYMBOL_ASK);                    

                    if (OrderSend(close_request, close_result))
                    {
                        Print("Ordine chiuso con successo. Ticket: ", close_result.order);
                    }
                    else
                    {
                        Print("Errore nella chiusura dell'ordine: ", _LastError);
                        Print("Codice errore: ", GetLastError());
                    }

                    orders_count--;
                }
            }
        }
    }
    else
    {
        // Controllo se si deve aprire una nuova posizione
        if (current_price > prev_price + Distance && orders_count < MaxOrders)
        {
            MqlTradeRequest open_request = { TRADE_ACTION_DEAL };
            MqlTradeResult open_result = {0};

            open_request.action = TRADE_ACTION_DEAL;
            open_request.symbol = _Symbol;
            open_request.volume = Lots;
            open_request.type = ORDER_TYPE_BUY;
            open_request.price = SymbolInfoDouble(open_request.symbol, SYMBOL_ASK);

            if (OrderSend(open_request, open_result))
            {
                Print("Ordine aperto con successo. Ticket: ", open_result.order);
                orders_count++;
            }
            else
            {
                Print("Errore nell'apertura dell'ordine: ", _LastError);
                Print("Codice errore: ", GetLastError());
            }
        }
        else if (current_price < prev_price - Distance && orders_count < MaxOrders)
        {
            MqlTradeRequest open_request = { TRADE_ACTION_DEAL };
            MqlTradeResult open_result = {0};

            open_request.action = TRADE_ACTION_DEAL;
            open_request.symbol = _Symbol;
            open_request.volume = Lots;
            open_request.type = ORDER_TYPE_SELL;
            open_request.price = SymbolInfoDouble(open_request.symbol, SYMBOL_BID);

            if (OrderSend(open_request, open_result))
            {
                Print("Ordine aperto con successo. Ticket: ", open_result.order);
                orders_count++;
        }
        else
        {
            Print("Errore nell'apertura dell'ordine: ", _LastError);
            Print("Codice errore: ", GetLastError());
        }
    }
}

prev_price = current_price;
}

//+------------------------------------------------------------------+
//| Funzione di inizializzazione |
//+------------------------------------------------------------------+
void OnInit()
{
Print("EA inizializzato");
}

//+------------------------------------------------------------------+
//| Funzione di deinizializzazione |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
Print("EA disattivato con motivo ", reason);
}

//+------------------------------------------------------------------+
//| Funzione per gestire gli errori |
//+------------------------------------------------------------------+
void OnError(const int error_code)
{
Print("Errore: ", error_code);
}
