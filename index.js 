// Discord.js v14 Bot Example (Erweitert + Kategorie Check)
// Features:
// - Button wird nach Klick deaktiviert
// - Button wird deaktiviert wenn User den Channel verlässt
// - Nach "Übernehmen" wird Embed zu "Schließen"
// - "Schließen" kickt User 1 aus Voice
// - User kann NUR in Voice-Channels einer bestimmten Kategorie verschoben werden

const { Client, GatewayIntentBits, EmbedBuilder, ActionRowBuilder, ButtonBuilder, ButtonStyle, Events, ChannelType } = require('discord.js');

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildVoiceStates
  ]
});

const TOKEN = "DEIN_BOT_TOKEN";
const CHANNEL_1_ID = "1483918907747340318"; // Warteschlange // Warteschlange
const CHANNEL_2_ID = "1484291035751514122";  // Embed Channel  // Embed Channel
const ALLOWED_CATEGORY_ID = "1477718723203961124"; // Erlaubte Kategorie // <<< WICHTIG

let waitingUser = null;
let messageRef = null;

client.on(Events.VoiceStateUpdate, async (oldState, newState) => {
  // User joint Channel 1
  if (newState.channelId === CHANNEL_1_ID && oldState.channelId !== CHANNEL_1_ID) {
    waitingUser = newState.member;

    const embed = new EmbedBuilder()
      .setTitle("Neuer User wartet")
      .setDescription(`${newState.member.user.tag} wartet im Channel`)
      .setColor(0x00ff00);

    const button = new ButtonBuilder()
      .setCustomId("claim_user")
      .setLabel("Übernehmen")
      .setStyle(ButtonStyle.Primary);

    const row = new ActionRowBuilder().addComponents(button);

    const channel = await client.channels.fetch(CHANNEL_2_ID);
    messageRef = await channel.send({ embeds: [embed], components: [row] });
  }

  // User verlässt Channel 1
  if (oldState.channelId === CHANNEL_1_ID && newState.channelId !== CHANNEL_1_ID) {
    if (messageRef) {
      const disabledRow = new ActionRowBuilder().addComponents(
        new ButtonBuilder()
          .setCustomId("claim_user")
          .setLabel("Übernehmen")
          .setStyle(ButtonStyle.Secondary)
          .setDisabled(true)
      );

      await messageRef.edit({ components: [disabledRow] });
    }

    waitingUser = null;
  }
});

client.on(Events.InteractionCreate, async interaction => {
  if (!interaction.isButton()) return;

  // Übernehmen
  if (interaction.customId === "claim_user") {
    if (!waitingUser) {
      return interaction.reply({ content: "Kein User vorhanden", ephemeral: true });
    }

    const member = interaction.member;
    const voiceChannel = member.voice.channel;

    if (!voiceChannel) {
      return interaction.reply({ content: "Du bist in keinem Voice Channel", ephemeral: true });
    }

    // ❗ Kategorie Check
    if (
      voiceChannel.type !== ChannelType.GuildVoice ||
      voiceChannel.parentId !== ALLOWED_CATEGORY_ID
    ) {
      return interaction.reply({
        content: "Du darfst nur Channels aus der erlaubten Kategorie verwenden!",
        ephemeral: true
      });
    }

    try {
      await waitingUser.voice.setChannel(voiceChannel);

      const embed = new EmbedBuilder()
        .setTitle("User übernommen")
        .setDescription(`${waitingUser.user.tag} ist jetzt bei ${member.user.tag}`)
        .setColor(0x0099ff);

      const closeButton = new ButtonBuilder()
        .setCustomId("close_user")
        .setLabel("Schließen")
        .setStyle(ButtonStyle.Danger);

      const row = new ActionRowBuilder().addComponents(closeButton);

      await interaction.update({ embeds: [embed], components: [row] });
    } catch (err) {
      console.error(err);
      interaction.reply({ content: "Fehler beim Verschieben", ephemeral: true });
    }
  }

  // Schließen
  if (interaction.customId === "close_user") {
    if (!waitingUser) {
      return interaction.reply({ content: "Kein User vorhanden", ephemeral: true });
    }

    try {
      await waitingUser.voice.disconnect();

      const embed = new EmbedBuilder()
        .setTitle("Session geschlossen")
        .setDescription("User wurde gekickt")
        .setColor(0xff0000);

      const disabledRow = new ActionRowBuilder().addComponents(
        new ButtonBuilder()
          .setCustomId("closed")
          .setLabel("Geschlossen")
          .setStyle(ButtonStyle.Secondary)
          .setDisabled(true)
      );

      waitingUser = null;

      await interaction.update({ embeds: [embed], components: [disabledRow] });
    } catch (err) {
      console.error(err);
      interaction.reply({ content: "Fehler beim Kicken", ephemeral: true });
    }
  }
});

client.login(TOKEN);
