import { useEffect, useState } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";

export default function Dashboard() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch("/data/cory_index.csv")
      .then((res) => res.text())
      .then((text) => {
        const rows = text.split("\n");
        const parsed = rows
          .map((row) => {
            const [date, price] = row.split(",");
            return {
              date,
              price: parseFloat(price),
            };
          })
          .filter((r) => r.date && !isNaN(r.price));

        setData(parsed);
      });
  }, []);

  const latest = data[data.length - 1]?.price || 0;
  const prev = data[data.length - 2]?.price || 0;
  const change = prev ? (((latest - prev) / prev) * 100).toFixed(2) : 0;

  return (
    <div className="p-6 grid gap-6">
      <h1 className="text-3xl font-bold">Corydoras Market Index</h1>

      <Card className="rounded-2xl shadow">
        <CardContent className="p-6">
          <div className="text-xl">Current Price</div>
          <div className="text-4xl font-bold">${latest}</div>
          <div className="text-sm mt-2">
            {change >= 0 ? "📈" : "📉"} {change}% (24h)
          </div>
        </CardContent>
      </Card>

      <Card className="rounded-2xl shadow">
        <CardContent className="p-6">
          <h2 className="text-xl mb-4">Price Trend</h2>
          <ResponsiveContainer width="100%" height={300}>
            <LineChart data={data}>
              <XAxis dataKey="date" />
              <YAxis />
              <Tooltip />
              <Line type="monotone" dataKey="price" strokeWidth={3} />
            </LineChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>
    </div>
  );
}
