#include "SessionAgregator.h"
#include "../settings/Settings.h"
#include "../db/DbConnection.h"
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <boost/lexical_cast.hpp>
#include "../utils/constants/DbCnst.h"
#include "../utils/constants/FieldCnst.h"
#include "../utils/constants/CollectionCnst.h"
#include "../utils/constants/UsersSettingsCnst.h"

используя bsoncxx::builder::basic::kvp;

std::map<std::string, Session> SessionAgregator::currentConnections;

bool SessionAgregator::sessionDead(std::string uuidForSession) {
    auto availableSession = currentConnections.find(uuidForSession);
    если (доступный сеанс != currentConnections.end()) {
        auto thisSession = currentConnections[uuidForSession];
        если (diffMoreTtl(thisSession.creationTime)) {
            // РµСЃР»Рё СЃРµСЃСЃРёСЏ РїСЂРѕС‚СѓС…Р»Р°, РІС‹РєРёРЅСѓС‚СЊ РµС' РёР· РјР°РїС‹
            currentConnections.erase(uuidForSession);
            вернуть истину;
        } еще {
            updateSessionTime (uuidForSession, thisSession);
            вернуть ложь;
        }
    } еще {
        // РµСЃР»Рё СЃРµСЃСЃРёРё СЃРѕРІСЃРµРј РЅРµС‚ РІ РјР°РїРµ, Р·РЅР°С‡РёС‚ РѕРЅР° РЅРµ СЃРѕР·РґР°РІР°Р»Р°СЃСЊ РёР»Рё РїСЂРѕС‚СѓС…Р»Р°
        вернуть истину;
    }
}

void SessionAgregator::updateSessionTime(const std::string &uuidForSession, Session &thisSession) {
    // РѕР±РЅРѕРІР»СЏРµРј СЃРµСЃСЃРёСЋ, С‚Рє. РїРѕСЃС‚СѓРїРёР» РЅРѕРІС‹Р№ Р·Р°РїСЂРѕСЃ
    thisSession.creationTime = getCurrentTime();
    // РєР»Р°РґС'Рј РѕР±РЅРѕРІР»С'РЅРЅРѕРµ Р·РЅР°С‡РµРЅРёРµ РІ РјР°РїСѓ
    currentConnections.erase(uuidForSession);
    currentConnections[uuidForSession] = thisSession;
}

bool SessionAgregator:: diffMoreTtl (время создания tm) {
    time_t секунд = время (NULL);
    tm *now = местное время (& секунд);
    auto diff = difftime(mktime(сейчас), mktime(&creationTime));
    вернуть разницу >= TTL;
}

Сессия SessionAgregator::getSessionById(std::string id) {
    вернуть текущие соединения [id];
}

std::string SessionAgregator::createSession(web::json::значение значения) {
    std::строка authInStr;
    auto userLogin = значение[FieldCnst::LOGIN].as_string();
    authInStr = returnSessionIfAlreadyExists(userLogin);
    если (!authInStr.empty()) {
        если (!sessionDead(authInStr)) {
            вернуть authInStr;
        }
    } еще {
        // РіРµРЅРµСЂРёРј СЋСЋРёРґ
        authInStr = generateUuid (authInStr);
        // Р·Р°РїРѕР»РЅСЏРµРј РїРѕР»СЏ РІ РјР°РїРµ
        Сеанс сеанса = getFieldsFromSession(userLogin);
        fillMap (authInStr, сеанс);
    }
    вернуть authInStr;
}

void SessionAgregator::fillMap(const std::string &authInStr, const Session &session) {
    currentConnections[authInStr].creationTime = session.creationTime;
    currentConnections[authInStr].login = session.login;
    currentConnections[authInStr].rights = session.rights;
}

const std::string SessionAgregator::returnSessionIfAlreadyExists(utility::string_t userLogin) {
    for (auto &connection : currentConnections) {
        если (userLogin == connection.second.login) {
            вернуть соединение.сначала;
        }
    }
    возврат "";
}


std::string &SessionAgregator::generateUuid(std::string &authInStr) {
    auto uuid = boost::uuids::random_generator();
    boost::uuids::uuid uuidAuth = boost::uuids::random_generator()();
    authInStr = boost::lexical_cast<std::__cxx11::string>(uuidAuth);
    вернуть authInStr;
}

Сессия SessionAgregator::getFieldsFromSession(std::string &userLogin) {
    Сессионный сеанс;
    session.creationTime = getCurrentTime();
    session.login = логин пользователя;
    session.rights = getUserRights (логин пользователя);
    обратная сессия;
}

tm SessionAgregator::getCurrentTime() {
    time_t секунд = время (NULL);
    tm timeinfo = *localtime(&seconds);
    вернуть информацию о времени;
}

Статус SessionAgregator::getUserRights(std::string &userLogin) {
    auto userRights = getUserStatusFromCollection (userLogin);
    вернуть UserStatus::getRightByStr(userRights);
}

std::string SessionAgregator::getUserStatusFromCollection(std::string &userLogin) {
    // Р'С‹С‡Р»РµРЅСЏРµРј СЃС‚Р°С‚СѓСЃ РёР· РєРѕР»Р»РµРєС†РёРё "РїСЂРѕС„РёР"СЊ"
    mongocxx::uri uri(Settings::getConnectionAuthString(UserSettingsCnst::ADMIN_LOGIN, UserSettingsCnst::ADMIN_PASSWORD));
    автоматический клиент = mongocxx::client(uri);
    mongocxx::v_noabi::database dbasedb = client[DbCnst::NAME];
    автоколлекция = dbasedb.collection(CollectionCnst::PROFILE);
    автоматический курсор = collection.find_one({getFilter(userLogin)});
    auto userRights = cursor->view()[FieldCnst::STATUS].get_utf8().value.to_string();
    вернуть права пользователя;
}

bsoncxx::builder::basic::document SessionAgregator::getFilter(std::string userLogin) {
    автоматический фильтр = bsoncxx::builder::basic::document{};
    filter.append(kvp(FieldCnst::LOGIN, userLogin.c_str()));
    возвратный фильтр;
}
