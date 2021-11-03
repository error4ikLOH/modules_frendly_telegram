# Admin Tools for Friendly-Telegram UserBot.
# Copyright (C) 2020 @Fl1yd, @Djdkxkxys
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.
# ======================================================================

import io, time
from .. import loader, utils, security
from PIL import Image
from telethon.errors import (ChatAdminRequiredError, UserAdminInvalidError, FloodWaitError, PhotoCropSizeSmallError)
from telethon.tl.types import (ChatAdminRights, ChatBannedRights)
from telethon.tl.functions.channels import (EditAdminRequest, EditBannedRequest, EditPhotoRequest, DeleteUserHistoryRequest)
from telethon.tl.functions.messages import EditChatAdminRequest

# ================== КОНСТАНТЫ ========================

PROMOTE_RIGHTS = ChatAdminRights(add_admins=False,
                                 invite_users=True,
                                 change_info=False,
                                 ban_users=True,
                                 delete_messages=True,
                                 pin_messages=True)

DEMOTE_RIGHTS = ChatAdminRights(post_messages=None,
                                add_admins=None,
                                invite_users=None,
                                change_info=None,
                                ban_users=None,
                                delete_messages=None,
                                pin_messages=None,
                                edit_messages=None)

UNMUTE_RIGHTS = ChatBannedRights(until_date=None,
                                 view_messages=None,
                                 send_messages=False,
                                 send_media=False,
                                 send_stickers=False,
                                 send_gifs=False,
                                 send_games=False,
                                 send_inline=False,
                                 embed_links=False)

BANNED_RIGHTS = ChatBannedRights(until_date=None,
                                 view_messages=True,
                                 send_messages=True,
                                 send_media=True,
                                 send_stickers=True,
                                 send_gifs=True,
                                 send_games=True,
                                 send_inline=True,
                                 embed_links=True)

UNBAN_RIGHTS = ChatBannedRights(until_date=None,
                                view_messages=None,
                                send_messages=None,
                                send_media=None,
                                send_stickers=None,
                                send_gifs=None,
                                send_games=None,
                                send_inline=None,
                                embed_links=None)

# =====================================================

def register(cb):
    cb(AdminToolsMod())

class AdminToolsMod(loader.Module):
    """Администрирование чата"""
    strings = {'name': 'AdminTools',
               'no_reply': '<b>Нет реплая на картинку/стикер.</b>',
               'not_pic': '<b>Это не картинка/стикер</b>',
               'wait': '<b>Минуточку...</b>',
               'pic_so_small': '<b>Картинка слишком маленькая, попробуйте другую.</b>',
               'pic_changed': '<b>Картинка чата изменена.</b>',
               'promote_none': '<b>Некого повышать.</b>',
               'who': '<b>Кто это?</b>',
               'not_admin': '<b>Я здесь не админ.</b>',
               'promoted': '<b>{} повышен в правах администратора.\nРанг: {}</b>',
               'wtf_is_it': '<b>Что это?</b>',
               'this_isn`t_a_chat': '<b>Это не чат!</b>',
               'demote_none': '<b>Некого понижать.</b>',
               'demoted': '<b>{} понижен в правах администратора.</b>',
               'pinning': '<b>Пин...</b>',
               'pin_none': '<b>Ответь на сообщение чтобы закрепить его.</b>',
               'unpinning': '<b>Анпин...</b>',
               'unpin_none': '<b>Нечего откреплять.</b>',
               'no_rights': '<b>У меня нет прав.</b>',
               'pinned': '<b>Закреплено успешно!</b>',
               'unpinned': '<b>Откреплено успешно!</b>',
               'can`t_kick': '<b>Не могу кикнуть пользователя.</b>',
               'kicking': '<b>Кик...</b>',
               'kick_none': '<b>Некого кикать.</b>',
               'kicked': '<b>{} кикнут из чата.</b>',
               'kicked_for_reason': '<b>{} кикнут из чата.\nПричина: {}.</b>',
               'banning': '<b>Бан...</b>',
               'banned': '<b>{} забанен в чате.</b>',
               'banned_for_reason': '<b>{} забанен в чате.\nПричина: {}</b>', 
               'ban_none': '<b>Некому давать бан.</b>',
               'unban_none': '<b>Некого разбанить.</b>',
               'unbanned': '<b>{} разбанен в чате.</b>',
               'mute_none': '<b>Некому давать мут.</b>',
               'muted': '<b>{} теперь в муте на </b>',
               'no_args': '<b>Неверно указаны аргументы.</b>',
               'unmute_none': '<b>Некого размутить.</b>',
               'unmuted': '<b>{} теперь не в муте.</b>',
               'no_reply': '<b>Нет реплая.</b>',
               'deleting': '<b>Удаление...</b>',
               'no_args_or_reply':'<b>Нет аргументов или реплая.</b>',
               'deleted': '<b>Все сообщения от {} удалены.</b>',
               'del_u_search': '<b>Поиск удалённых аккаунтов...</b>',
               'del_u_kicking': '<b>Кик удалённых аккаунтов...\nОх~, я могу это сделать?!</b>'}


    async def ecpcmd(self, message):
        """Команда .ecp изменяет картинку чата.\nИспользование: .ecp <реплай на картинку/стикер>."""
        if message.chat:
            try:
                reply = await message.get_reply_message()
                chat = await message.get_chat()
                if not chat.admin_rights and not chat.creator:
                    return await utils.answer(message, self.strings('not_admin', message))
                if reply:
                    pic = await check_media(message, reply)
                    if not pic:
                        return await utils.answer(message, self.strings('not_pic', message))
                else:
                    return await utils.answer(message, self.strings('no_reply', message))
                await utils.answer(message, self.strings('wait', message))
                what = resizepic(pic)
                if what:
                    try:
                        await message.client(EditPhotoRequest(message.chat_id, await message.client.upload_file(what)))
                    except PhotoCropSizeSmallError:
                        return await utils.answer(message, self.strings('pic_so_small', message))
                await utils.answer(message, self.strings('pic_changed', message))
            except ChatAdminRequiredError:
                return await utils.answer(message, self.strings('no_rights', message))
        else:
            return await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def promotecmd(self, message):
        """Команда .promote повышает пользователя в правах администратора.\nИспользование: .promote <@ или реплай> <ранг>."""
        if message.chat:
            try:
                args = utils.get_args_raw(message).split(' ')
                reply = await message.get_reply_message()
                rank = 'одмэн'
                chat = await message.get_chat()
                adm_rights = chat.admin_rights 
                if not adm_rights and not chat.creator:
                    return await utils.answer(message, self.strings('not_admin', message))
                if reply:
                    args = utils.get_args_raw(message)
                    if args: rank = args
                    else: rank = rank
                    user = await message.client.get_entity(reply.sender_id)
                else:
                    user = await message.client.get_entity(args[0] if not args[0].isnumeric() else int(args[0]))
                    if len(args) == 1:
                        rank = rank
                    elif len(args) >= 2:
                        rank = utils.get_args_raw(message).split(' ', 1)[1]
                try:
                    await message.client(EditAdminRequest(message.chat_id, user.id, ChatAdminRights(add_admins=False, invite_users=adm_rights.invite_users,
                                                                                                    change_info=False, ban_users=adm_rights.ban_users,
                                                                                                    delete_messages=adm_rights.delete_messages, pin_messages=adm_rights.pin_messages), rank))
                except ChatAdminRequiredError:
                    return await utils.answer(message, self.strings('no_rights', message))
                else:
                    return await utils.answer(message, self.strings('promoted', message).format(user.first_name, rank))
            except ValueError:
                return await utils.answer(message, self.strings('no_args', message))
        else:
            return await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def demotecmd(self, message):
        """Команда .demote понижает пользователя в правах администратора.\nИспользование: .demote <@ или реплай>."""
        if not message.is_private:
            try:
                reply = await message.get_reply_message()
                chat = await message.get_chat()
                if not chat.admin_rights and not chat.creator:
                    return await utils.answer(message, self.strings('not_admin', message))
                if reply:
                    user = await message.client.get_entity(reply.sender_id)
                else:
                    args = utils.get_args_raw(message)
                    if not args:
                        return await utils.answer(message, self.strings('demote_none', message))
                    user = await message.client.get_entity(args if not args.isnumeric() else int(args))
                try:
                    if message.is_channel:
                        await message.client(EditAdminRequest(message.chat_id, user.id, DEMOTE_RIGHTS, ""))
                    else:
                        await message.client(EditChatAdminRequest(message.chat_id, user.id, False))
                except ChatAdminRequiredError:
                    return await utils.answer(message, self.strings('no_rights', message))
                else:
                    return await utils.answer(message, self.strings('demoted', message).format(user.first_name))
            except ValueError:
                return await utils.answer(message, self.strings('no_args'))
        else:
            return await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def pincmd(self, message):
        """Команда .pin закрепляет сообщение в чате.\nИспользование: .pin <реплай>."""
        if not message.is_private:
            reply = await message.get_reply_message()
            if not reply:
                return await utils.answer(message, self.strings('pin_none', message))
            await utils.answer(message, self.strings('pinning', message))
            try:
                await message.client.pin_message(message.chat, message=reply.id, notify=False)
            except ChatAdminRequiredError:
                return await utils.answer(message, self.strings('no_rights', message))
            await utils.answer(message, self.strings('pinned', message))
        else:
            await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def unpincmd(self, message):
        """Команда .unpin открепляет закрепленное сообщение в чате.\nИспользование: .unpin."""
        if not message.is_private:
            await utils.answer(message, self.strings('unpinning', message))
            try:
                await message.client.pin_message(message.chat, message=None, notify=None)
            except ChatAdminRequiredError:
                return await utils.answer(message, self.strings('no_rights', message))
            await utils.answer(message, self.strings('unpinned', message))
        else:
            await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def kickcmd(self, message):
        """Команда .kick кикает пользователя.\nИспользование: .kick <@ или реплай>."""
        if not message.is_private:
            try:
                args = utils.get_args_raw(message).split(' ')
                reason = utils.get_args_raw(message)
                reply = await message.get_reply_message()
                chat = await message.get_chat()
                if not chat.admin_rights and not chat.creator:
                    return await utils.answer(message, self.strings('not_admin', message))
                else:
                    if chat.admin_rights.ban_users == False:
                        return await utils.answer(message, self.strings('no_rights', message))
                if reply:
                    user = await message.client.get_entity(reply.sender_id)
                    args = utils.get_args_raw(message)
                    if args:
                        reason = args
                else:
                    user = await message.client.get_entity(args[0] if not args[0].isnumeric() else int(args[0]))
                    if args:
                        if len(args) == 1:
                            args = utils.get_args_raw(message)
                            user = await message.client.get_entity(args if not args.isnumeric() else int(args))
                            reason = False
                        elif len(args) >= 2:
                            reason = utils.get_args_raw(message).split(' ', 1)[1]
                await utils.answer(message, self.strings('kicking', message))
                try:
                    await message.client.kick_participant(message.chat_id, user.id)
                except UserAdminInvalidError:
                    return await utils.answer(message, self.strings('no_rights', message))
                if reason == False:
                    return await utils.answer(message, self.strings('kicked', message).format(user.first_name))
                if reason:
                    return await utils.answer(message, self.strings('kicked_for_reason', message).format(user.first_name, reason))
                return await utils.answer(message, self.strings('kicked', message).format(user.first_name))
            except ValueError:
                return await utils.answer(message, self.strings('no_args', message))
        else:
            return await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def bancmd(self, message):
        """Команда .ban даёт бан пользователю.\nИспользование: .ban <@ или реплай>."""
        if not message.is_private:
            try:
                args = utils.get_args_raw(message).split(' ')
                reason = utils.get_args_raw(message)
                reply = await message.get_reply_message()
                chat = await message.get_chat()
                if not chat.admin_rights and not chat.creator:
                    return await utils.answer(message, self.strings('not_admin', message))
                else:
                    if chat.admin_rights.ban_users == False:
                        return await utils.answer(message, self.strings('no_rights', message))
                if reply:
                    user = await message.client.get_entity(reply.sender_id)
                    args = utils.get_args_raw(message)
                    if args:
                        reason = args
                else:
                    user = await message.client.get_entity(args[0] if not args[0].isnumeric() else int(args[0]))
                    if args:
                        if len(args) == 1:
                            args = utils.get_args_raw(message)
                            user = await message.client.get_entity(args if not args.isnumeric() else int(args))
                            reason = False
                        elif len(args) >= 2:
                            reason = utils.get_args_raw(message).split(' ', 1)[1]
                try:
                    await utils.answer(message, self.strings('banning', message))
                    await message.client(EditBannedRequest(message.chat_id, user.id, ChatBannedRights(until_date=None, view_messages=True)))
                except UserAdminInvalidError:
                    return await utils.answer(message, self.strings('no_rights', message))
                if reason == False:
                    return await utils.answer(message, self.strings('banned', message).format(user.first_name))
                if reason:
                    return await utils.answer(message, self.strings('banned_for_reason', message).format(user.first_name, reason))
                return await utils.answer(message, self.strings('banned', message).format(user.first_name))
            except ValueError:
                return await utils.answer(message, self.strings('no_args', message))
        else:
            return await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def unbancmd(self, message):
        """Команда .unban для разбана пользователя.\nИспользование: .unban <@ или реплай>."""
        if not message.is_private:
            try:
                reply = await message.get_reply_message() 
                chat = await message.get_chat()
                if not chat.admin_rights and not chat.creator:
                    return await utils.answer(message, self.strings('not_admin', message))
                else:
                    if chat.admin_rights.ban_users == False:
                        return await utils.answer(message, self.strings('no_rights', message))
                if reply:
                    user = await message.client.get_entity(reply.sender_id)
                else:
                    args = utils.get_args_raw(message)
                    if not args:
                        return await utils.answer(message, self.strings('unban_none', message))
                    user = await message.client.get_entity(args if not args.isnumeric() else int(args))
                await message.client(EditBannedRequest(message.chat_id, user.id, ChatBannedRights(until_date=None, view_messages=False)))
                return await utils.answer(message, self.strings('unbanned', message).format(user.first_name))
            except ValueError:
                return await utils.answer(message, self.strings('no_args', message))
        else:
            return await utils.answer(message, self.strings('this_isn`t_a_chat', message))


    async def mutecmd(self, message):
        """Команда .mute даёт мут пользователю.\nИспользование: .mute <@ или реплай> <время (1m, 1h, 1d)>."""
        if not message.is_private:
            try:
                reply = await message.get_reply_message()
                chat = await message.get_chat()
                if not chat.admin_rights and not chat.creator:
                    return await utils.answer(message, self.strings('not_admin', message))
                else:
                    if chat.admin_rights.ban_users  
