import React, { useState, useEffect } from "react";
import { motion } from "framer-motion";
import Quagga from "quagga";

// Simple in-memory store (replace with API/backend later)
const initialDB = {
  "123456": {
    name: "Cabo HDMI",
    price: 29.9,
    type: "unidade",
    stock: 12,
  },
};

export default function InventoryApp() {
  const [db, setDb] = useState(initialDB);
  const [code, setCode] = useState("");
  const [result, setResult] = useState(null);
  const [scanning, setScanning] = useState(false);

  // Barcode scanner
  useEffect(() => {
    if (!scanning) return;

    Quagga.init(
      {
        inputStream: {
          name: "Live",
          type: "LiveStream",
          target: document.querySelector("#scanner"),
          constraints: { facingMode: "environment" },
        },
        decoder: { readers: ["ean_reader", "code_128_reader"] },
      },
      (err) => {
        if (err) return;
        Quagga.start();
      }
    );

    Quagga.onDetected((data) => {
      const scanned = data.codeResult.code;
      setCode(scanned);
      search(scanned);
      setScanning(false);
      Quagga.stop();
    });

    return () => {
      Quagga.stop();
      Quagga.offDetected();
    };
  }, [scanning]);

  function search(c) {
    if (db[c]) setResult(db[c]);
    else setResult("NOT_FOUND");
  }

  function registerProduct() {
    const name = prompt("Nome do produto:");
    const price = parseFloat(prompt("Preço:") || 0);
    const type = prompt("Tipo (metro / par / unidade):");
    const stock = parseFloat(prompt("Estoque:") || 0);

    if (!name || !type) return;

    setDb((prev) => ({
      ...prev,
      [code]: { name, price, type, stock },
    }));
    alert("Produto cadastrado!");
  }

  return (
    <div className="min-h-screen bg-gray-100 p-4">
      {/* Header */}
      <h1 className="text-3xl font-bold text-center mb-6">Sistema de Estoque — Mobile</h1>

      {/* Input */}
      <div className="max-w-md mx-auto bg-white p-4 rounded-xl shadow">
        <label className="text-sm font-semibold">Código do Produto</label>
        <input
          value={code}
          onChange={(e) => setCode(e.target.value)}
          className="w-full p-3 border rounded mt-2"
          placeholder="Digite ou escaneie o código"
        />

        <div className="mt-4 flex gap-3">
          <button
            className="flex-1 bg-indigo-600 text-white p-3 rounded-lg"
            onClick={() => search(code)}
          >
            Consultar
          </button>
          <button
            className="flex-1 bg-green-600 text-white p-3 rounded-lg"
            onClick={() => setScanning(true)}
          >
            📷 Escanear
          </button>
        </div>
      </div>

      {/* Scanner View */}
      {scanning && (
        <div className="mt-6 max-w-md mx-auto bg-black rounded-lg overflow-hidden">
          <div id="scanner" className="w-full h-72" />
        </div>
      )}

      {/* Result Box */}
      {result && (
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          className="max-w-md mx-auto mt-6 bg-white p-4 rounded-xl shadow"
        >
          {result === "NOT_FOUND" ? (
            <>
              <p className="text-red-600 font-semibold">Produto não encontrado.</p>
              <button
                className="mt-3 w-full bg-blue-600 text-white p-3 rounded-lg"
                onClick={registerProduct}
              >
                Cadastrar produto
              </button>
            </>
          ) : (
            <>
              <h2 className="text-xl font-bold">{result.name}</h2>
              <p className="mt-2">Preço: R$ {result.price.toFixed(2)}</p>
              <p>Tipo: {result.type}</p>
              <p>Estoque: {result.stock}</p>
            </>
          )}
        </motion.div>
      )}
    </div>
  );
}
