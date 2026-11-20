# Auto-Deco-mod
New mode for Auto-Deco using neon
#include <Geode/Geode.hpp>
#include <Geode/modify/EditorUI.hpp>
#include <Geode/ui/Popup.hpp>

using namespace geode::prelude;

void setupNeonColors(LevelEditorLayer* editorLayer, int mode) {
    if (!editorLayer || !editorLayer->m_levelSettings) return;

    auto effectManager = editorLayer->m_levelSettings->m_effectManager;
    if (!effectManager) return;

    if (auto color1 = effectManager->getColorAction(1)) {
        color1->m_fromColor = mode == 0 ? ccc3(180, 0, 255) : ccc3(0, 140, 255);
        color1->m_toColor = color1->m_fromColor;
    }

    if (auto color2 = effectManager->getColorAction(2)) {
        color2->m_fromColor = mode == 0 ? ccc3(0, 180, 255) : ccc3(190, 0, 255);
        color2->m_toColor = color2->m_fromColor;
    }

    if (auto color3 = effectManager->getColorAction(3)) {
        color3->m_fromColor = mode == 0 ? ccc3(210, 80, 255) : ccc3(80, 210, 255);
        color3->m_toColor = color3->m_fromColor;
        color3->m_blending = true;
    }
}

class NeonDecorPopup : public Popup<EditorUI*> {
protected:
    EditorUI* m_editorUI = nullptr;

    bool setup(EditorUI* editorUI) override {
        m_editorUI = editorUI;
        setTitle("Neon AutoDecor");

        auto winSize = m_mainLayer->getContentSize();

        auto info = CCLabelBMFont::create(
            "Vydeli bloki i shipy v redaktore:",
            "bigFont.fnt"
        );
        info->setScale(0.45f);
        info->setPosition({
            winSize.width / 2.f,
            winSize.height - 55.f
        });
        m_mainLayer->addChild(info);

        auto purpleButton = CCMenuItemSpriteExtra::create(
            ButtonSprite::create(
                "Neon: Fioletovo-Siniy",
                "goldFont.fnt",
                "GJ_button_01.png",
                0.8f
            ),
            this,
            menu_selector(NeonDecorPopup::onApplyPurpleNeon)
        );
        purpleButton->setPosition({
            winSize.width / 2.f,
            winSize.height / 2.f + 15.f
        });

        auto blueButton = CCMenuItemSpriteExtra::create(
            ButtonSprite::create(
                "Neon: Sine-Fioletoviy",
                "goldFont.fnt",
                "GJ_button_02.png",
                0.8f
            ),
            this,
            menu_selector(NeonDecorPopup::onApplyBlueNeon)
        );
        blueButton->setPosition({
            winSize.width / 2.f,
            winSize.height / 2.f - 40.f
        });

        auto menu = CCMenu::create();
        menu->setPosition({0.f, 0.f});
        menu->addChild(purpleButton);
        menu->addChild(blueButton);
        m_mainLayer->addChild(menu);

        return true;
    }

    void applyStyle(int mode) {
        if (!m_editorUI) return;

        auto selected = m_editorUI->getSelectedObjects();
        if (!selected || selected->count() == 0) {
            FLAlertLayer::create(
                "Oshibka",
                "Snachala vydeli bloki ili shipy!",
                "OK"
            )->show();
            return;
        }

        auto editorLayer = m_editorUI->m_editorLayer;
        if (!editorLayer) return;

        setupNeonColors(editorLayer, mode);

        CCArray* newObjects = CCArray::create();

        for (auto obj : CCArrayExt<GameObject*>(selected)) {
            if (!obj) continue;

            const auto pos = obj->getPosition();
            const float rotation = obj->getRotation();
            const float scaleX = obj->getScaleX();
            const float scaleY = obj->getScaleY();

            int newID = obj->m_objectID;

            if (obj->m_objectID == 1) {
                newID = 211;
            } else if (obj->m_objectID == 8) {
                newID = 39;
            } else if (obj->m_objectID == 39) {
                newID = 103;
            }

            auto newObject = editorLayer->createObject(newID, pos, false);
            if (!newObject) continue;

            newObject->setRotation(rotation);
            newObject->setScaleX(scaleX);
            newObject->setScaleY(scaleY);

            if (newObject->m_baseColor) {
                newObject->m_baseColor->m_colorID = 1;
            }

            if (newObject->m_detailColor) {
                newObject->m_detailColor->m_colorID = 2;
            }

            auto glow = editorLayer->createObject(1753, pos, false);
            if (glow) {
                glow->setRotation(rotation);
                glow->setScaleX(scaleX * 1.15f);
                glow->setScaleY(scaleY * 1.15f);
                glow->setZOrder(newObject->getZOrder() - 1);

                if (glow->m_baseColor) {
                    glow->m_baseColor->m_colorID = 3;
                }

                newObjects->addObject(glow);
            }

            newObjects->addObject(newObject);
        }

        m_editorUI->deleteSelectedObjects(false);
        m_editorUI->selectObjects(newObjects, false);
        m_editorUI->updateButtons();

        onClose(nullptr);

        Notification::create(
            "Neon AutoDecor primenen!",
            NotificationIcon::Success
        )->show();
    }

    void onApplyPurpleNeon(CCObject*) {
        applyStyle(0);
    }

    void onApplyBlueNeon(CCObject*) {
        applyStyle(1);
    }

public:
    static NeonDecorPopup* create(EditorUI* editorUI) {
        auto popup = new NeonDecorPopup();

        if (popup && popup->initAnchored(320.f, 190.f, editorUI)) {
            popup->autorelease();
            return popup;
        }

        CC_SAFE_DELETE(popup);
        return nullptr;
    }
};

class $modify(NeonEditorUI, EditorUI) {
    bool init(LevelEditorLayer* editorLayer) {
        if (!EditorUI::init(editorLayer)) return false;

        if (auto menu = getChildByID("editor-buttons-menu")) {
            auto sprite = ButtonSprite::create(
                "NEON",
                "goldFont.fnt",
                "GJ_button_03.png",
                0.65f
            );

            auto button = CCMenuItemSpriteExtra::create(
                sprite,
                this,
                menu_selector(NeonEditorUI::onOpenNeonMenu)
            );

            button->setID("neon-autodecor-btn"_spr);
            menu->addChild(button);
            menu->updateLayout();
        }

        return true;
    }

    void onOpenNeonMenu(CCObject*) {
        NeonDecorPopup::create(this)->show();
    }
};
