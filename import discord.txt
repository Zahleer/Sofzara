import discord
from discord.ext import commands
import os
from flask import Flask
from threading import Thread
import asyncio

# Keep-alive simple
app = Flask(__name__)

@app.route("/")
def home():
    return "Bot vivo"

def run_flask():
    app.run(host="0.0.0.0", port=5000)

# Bot
intents = discord.Intents.default()
intents.voice_states = True
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f"{bot.user} conectado y listo!")

    # Keepalive básico (envía paquete vacío cada 15s)
    async def keepalive():
        while True:
            for vc in bot.voice_clients:
                if vc.is_connected():
                    try:
                        vc.send_audio_packet(b'\xF8\xFF\xFE', encode=False)
                    except:
                        pass
            await asyncio.sleep(15)

    bot.loop.create_task(keepalive())

@bot.command(name="unirse")
async def join_voice(ctx):
    if not ctx.author.voice:
        await ctx.send("¡Debes estar en voz!")
        return

    channel = ctx.author.voice.channel
    await channel.connect()
    await ctx.send(f"Entré a **{channel.name}** y me quedo (keepalive activo).")

@bot.command(name="salir")
async def leave_voice(ctx):
    if ctx.voice_client:
        await ctx.voice_client.disconnect()
        await ctx.send("Salí.")
    else:
        await ctx.send("No estoy en voz.")

if __name__ == "__main__":
    Thread(target=run_flask).start()
    bot.run(os.getenv("TOKEN"))
